<script lang="ts">
	import '../app.css';
	import { ModeWatcher } from 'mode-watcher';
	import { Toaster } from '$lib/components/ui/sonner';
</script>

<Toaster position="top-right" />
<ModeWatcher />
<div class="h-screen w-full">
	<slot />
</div>
