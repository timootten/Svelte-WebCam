<script lang="ts">
	import '../app.css';
	import { ModeWatcher } from 'mode-watcher';
</script>

<ModeWatcher />
<div class="h-screen w-full">
	<slot />
</div>
