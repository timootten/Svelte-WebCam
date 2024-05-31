<script lang="ts">
	import { browser } from '$app/environment';
	import { ArrowBigLeft } from 'lucide-svelte';
	import ThemeSwitcher from '$lib/components/core/ThemeSwitcher.svelte';
	import { Button } from '$lib/components/ui/button/index.js';

	function backPage() {}
</script>

<div class="absolute left-5 top-5 z-10 flex gap-3">
	<Button href="/" size="icon"><ArrowBigLeft /></Button>
	<ThemeSwitcher />
</div>
<slot />
