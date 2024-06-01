<script lang="ts">
	import Button from '$lib/components/ui/button/button.svelte';
	import * as Card from '$lib/components/ui/card/index';
	import SvelteOtp from '$lib/components/ui/otp/SvelteOtp.svelte';
	import * as Select from '$lib/components/ui/select';
	import { generateId } from '$lib/utils.js';
	import type { MediaConnection } from 'peerjs';
	import type Peer from 'peerjs';
	import { onMount } from 'svelte';
	import { toast } from 'svelte-sonner';

	let inputId = $state('');
	let peer = $state<Peer>();
	let mediaConnection = $state<MediaConnection>();
	let video = $state<HTMLVideoElement>();
	let isConnected = $state(false);
	let cameras = $state<MediaDeviceInfo[]>([]);
	let selectedCamera = $state<{ value: string; label: string }>();
	let mediaStream = $state<MediaStream>();

	const { data } = $props();

	onMount(() => {
		(async () => {
			await fetchCameras();

			const Peer = (await import('peerjs')).default;

			peer = new Peer(`${data.SECRET_KEY}-${generateId()}`);

			peer.on('open', function (id) {
				console.log('My peer ID is: ' + id);
			});
		})();
		return () => {
			peer?.destroy();
		};
	});

	const fetchCameras = async () => {
		try {
			await navigator.mediaDevices.getUserMedia({
				video: {
					deviceId: selectedCamera?.value,
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
			let frontCamera: MediaDeviceInfo | null = null;
			for (const device of cameras) {
				if (device.label.toLocaleLowerCase().includes('front')) {
					frontCamera = device;
					break;
				}
			}
			selectedCamera = frontCamera
				? { value: frontCamera.deviceId, label: frontCamera.label }
				: { value: cameras[0].deviceId, label: cameras[0].label };
		}

		mediaStream = await navigator.mediaDevices.getUserMedia({
			video: {
				deviceId: selectedCamera?.value,
				width: { ideal: 1920 },
				height: { ideal: 1080 }
			}
		});
		if (video) video.srcObject = mediaStream;
	};

	const handleCameraChange = async (value: string) => {
		//alert('XX' + value);

		mediaStream = await navigator.mediaDevices.getUserMedia({
			video: { deviceId: value, width: { ideal: 1920 }, height: { ideal: 1080 } }
		});
		if (video) video.srcObject = mediaStream;
		mediaConnection?.peerConnection.getSenders().forEach((sender) => {
			if (mediaStream instanceof MediaStream) {
				sender.replaceTrack(mediaStream.getVideoTracks()[0]);
			}
		});
	};

	const startWebCam = async (id: string) => {
		if (mediaStream) {
			peer?.on('error', (error) => {
				toast.error('The screen device code is invalid');
				isConnected = false;
			});
			mediaConnection = peer?.call(`${data.SECRET_KEY}-${id}`, mediaStream);
			mediaConnection?.on('close', () => {
				toast.error('The screen session has been closed.');
				isConnected = false;
			});
			mediaConnection?.on('willCloseOnRemote', () => {
				toast.success('Successfully connected to the screen.');
				isConnected = true;
			});
		}
	};

	$effect(() => {
		if (video && mediaStream) {
			video.srcObject = mediaStream;
			video.play();
		}
	});
</script>

<!-- svelte-ignore state_referenced_locally -->
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
					inputClass="rounded-xl text-xl text-center"
					inputStyle="width: 35px; height: 50px;"
					onSubmit={() => {
						startWebCam(inputId);
						inputId = '';
					}}
				/>
			</Card.Content>
		</Card.Root>
	</div>
{:else}
	<div
		class="grid h-screen w-full grid-cols-1 grid-rows-3 items-center gap-x-0 gap-y-5 p-5 md:grid-cols-3 md:grid-rows-1 md:gap-x-5 md:gap-y-0"
	>
		<video
			bind:this={video}
			autoplay
			playsinline
			muted
			class=" col-span-1 row-span-2 h-full w-full scale-x-[-1] bg-black md:col-span-2 md:row-span-1"
		>
			<track kind="captions" />
		</video>
		<Card.Root class="h-full w-full">
			<Card.Header>
				<Card.Title class="text-2xl">ShadeHost - Cam</Card.Title>
				<Card.Description>Information:</Card.Description>
			</Card.Header>
			<Card.Content>
				<div class="flex h-full w-full flex-col items-start gap-3">
					<Select.Root
						selected={selectedCamera}
						onSelectedChange={(x) => {
							if (x?.value) handleCameraChange(x?.value);
						}}
					>
						<Select.Trigger class="w-full">
							<Select.Value placeholder="Theme" />
						</Select.Trigger>
						<Select.Content>
							{#each cameras as camera (camera.deviceId)}
								<Select.Item value={camera.deviceId}
									>{camera.label || `Camera ${camera.deviceId}`}</Select.Item
								>
							{/each}
						</Select.Content>
					</Select.Root>
				</div>
			</Card.Content>
		</Card.Root>
	</div>
{/if}
