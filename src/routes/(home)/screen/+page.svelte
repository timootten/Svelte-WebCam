<script lang="ts">
	import * as Card from '$lib/components/ui/card/index';
	import SvelteOtp from '$lib/components/ui/otp/SvelteOtp.svelte';
	import { generateId } from '$lib/utils.js';
	import type { MediaConnection, Peer } from 'peerjs';
	import { onMount } from 'svelte';
	import { toast } from 'svelte-sonner';

	let peer = $state<Peer>();
	const id = $derived(peer?.id.split('-')[1]);
	let mediaConnection = $state<MediaConnection>();
	let video = $state<HTMLVideoElement>();
	let isConnected = $state(false);

	const { data } = $props();

	onMount(() => {
		(async () => {
			const Peer = (await import('peerjs')).default;

			peer = new Peer(`${data.SECRET_KEY}-${generateId()}`);

			peer.on('open', function (id) {
				console.log('My peer ID is: ' + id);
			});

			peer.on('call', (call) => {
				if (call.peer.split('-')[0] !== data.SECRET_KEY) return;
				toast.success('Successfully connected to the camera.');
				call.answer();
				isConnected = true;
				mediaConnection = call;
				call.on('close', () => {
					console.log('close');
					isConnected = false;
					toast.error('The camera session has been closed.');
				});
			});
		})();
		return () => {
			peer?.destroy();
		};
	});

	$effect(() => {
		if (video) {
			mediaConnection?.on('stream', (stream) => {
				console.log(stream);
				video!.srcObject = stream;
				video?.play();
			});
      mediaConnection?.options.
		}
	});
</script>

{#if !isConnected}
	<div class="flex h-screen items-center justify-center">
		<Card.Root class="mx-auto w-96 max-w-md">
			<Card.Header>
				<Card.Title class="text-2xl">ShadeHost - Screen</Card.Title>
				<Card.Description>Enter this code on your webcam device</Card.Description>
			</Card.Header>
			<Card.Content>
				<SvelteOtp
					value={id || ''}
					disabled={true}
					separator="-"
					separatorClass="font-bold"
					onlyShowMiddleSeparator={true}
					wrapperClass="w-full disabled"
					inputClass="rounded-xl disabled text-2xl font-bold"
					inputStyle="width: 45px; height: 60px;"
				/>
			</Card.Content>
		</Card.Root>
	</div>
{:else}
	<video
		id="video"
		bind:this={video}
		autoplay
		playsinline
		muted
		class="absolute left-0 top-0 z-20 h-screen w-full scale-x-[-1] bg-black"
	>
		<track kind="captions" />
	</video>
{/if}
