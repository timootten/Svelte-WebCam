<script lang="ts">
	import { Button } from '$lib/components/ui/button/index';
	import * as Card from '$lib/components/ui/card/index';
</script>

<div class="flex h-screen items-center justify-center">
	<Card.Root class="mx-auto max-w-sm">
		<Card.Header>
			<Card.Title class="text-2xl">ShadeHost - WebCam</Card.Title>
			<Card.Description>Select what this device should be</Card.Description>
		</Card.Header>
		<Card.Content>
			<div class="grid gap-4">
				<Button class="w-full" href="/screen">Screen</Button>
				<Button variant="outline" class="w-full" href="/cam">Camera</Button>
			</div>
		</Card.Content>
	</Card.Root>
</div>
