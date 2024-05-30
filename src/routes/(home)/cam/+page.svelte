<script lang="ts">
	import * as Card from '$lib/components/ui/card/index';
	import SvelteOtp from '$lib/components/ui/otp/SvelteOtp.svelte';
	import { generateId } from '$lib/utils.js';
	import type { MediaConnection } from 'peerjs';
	import type Peer from 'peerjs';
	import { onMount } from 'svelte';

	let inputId = $state('');
	let peer = $state<Peer>();
	let mediaConnection = $state<MediaConnection>();
	const id = $derived(peer?.id.split('-')[1]);
	let video = $state<HTMLVideoElement>();
	let isConnected = $state(false);
	let cameras = $state<MediaDeviceInfo[]>([]);
	let selectedCamera = $state<MediaDeviceInfo>();
	let mediaStream = $state<MediaStream>();

	const { data } = $props();

	onMount(async () => {
		await fetchCameras();

		const Peer = (await import('peerjs')).default;

		peer = new Peer(`${data.SECRET_KEY}-${generateId()}`);

		peer.on('open', function (id) {
			console.log('My peer ID is: ' + id);
		});
	});

	const fetchCameras = async () => {
		try {
			await navigator.mediaDevices.getUserMedia({
				video: {
					deviceId: selectedCamera?.deviceId,
					width: { ideal: 1920 },
					height: { ideal: 1080 }
				}
			});
		} catch (error) {
			alert(error);
		}
		const devices = await navigator.mediaDevices.enumerateDevices();
		cameras = devices.filter((device) => device.kind === 'videoinput');
		if (cameras.length > 0) {
			let backCamera: MediaDeviceInfo | null = null;
			for (const device of cameras) {
				if (
					device.label.toLocaleLowerCase().includes('back') ||
					device.label.toLocaleLowerCase().includes('rück')
				) {
					backCamera = device;
					break;
				}
			}
			selectedCamera = backCamera ? backCamera : cameras[0];
		}

		mediaStream = await navigator.mediaDevices.getUserMedia({
			video: {
				deviceId: selectedCamera?.deviceId,
				width: { ideal: 1920 },
				height: { ideal: 1080 }
			}
		});
		if (video) video.srcObject = mediaStream;
	};

	const handleCameraChange = async () => {
		const selectedIndex = (document.getElementById('cameraSelector') as HTMLSelectElement)
			.selectedIndex;
		selectedCamera = cameras[selectedIndex];

		mediaStream = await navigator.mediaDevices.getUserMedia({
			video: { deviceId: selectedCamera.deviceId, width: { ideal: 1920 }, height: { ideal: 1080 } }
		});
		if (video) video.srcObject = mediaStream;
		mediaConnection?.peerConnection.getSenders().forEach((sender) => {
			if (mediaStream instanceof MediaStream) {
				sender.replaceTrack(mediaStream.getVideoTracks()[0]);
			}
		});
	};

	const startWebCam = async (id: string) => {
		console.log('XXX');
		if (mediaStream) {
			isConnected = true;
			mediaConnection = peer?.call(`${data.SECRET_KEY}-${id}`, mediaStream);
			mediaConnection?.on('error', (error) => {
				console.log('Error', error);
			});
			mediaConnection?.on('stream', (stream) => {
				console.log('Stream', stream);
			});
		}
	};

	$effect(() => {
		console.log(inputId);
		if (inputId.length === 6) {
			$untrack();
			console.log('XX');
		}
	});
</script>

{#if !isConnected}
	<div class="flex h-screen items-center justify-center">
		<Card.Root class="mx-auto max-w-md">
			<Card.Header>
				<Card.Title class="text-2xl">ShadeHost - Cam</Card.Title>
				<Card.Description>Enter your code from the screen device</Card.Description>
			</Card.Header>
			<Card.Content>
				<SvelteOtp
					bind:value={inputId}
					separator="-"
					separatorClass="font-bold"
					onlyShowMiddleSeparator={true}
					wrapperClass="w-full"
					inputClass="rounded-xl text-xl"
					inputStyle="width: 35px; height: 50px;"
				/>
			</Card.Content>
		</Card.Root>
	</div>
{:else}
	<div
		class="grid h-screen w-full grid-cols-1 grid-rows-3 items-center gap-x-0 gap-y-5 p-5 md:grid-cols-3 md:grid-rows-1 md:gap-x-5 md:gap-y-0"
	>
		<Card.Root class="h-full w-full">
			<Card.Header>
				<Card.Title class="text-2xl">ShadeHost - Cam</Card.Title>
				<Card.Description>Information:</Card.Description>
			</Card.Header>
			<Card.Content>
				<p class="text-gray-600">SOON</p>
			</Card.Content>
		</Card.Root>
	</div>
{/if}
