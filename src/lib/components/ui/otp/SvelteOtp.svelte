<script lang="ts">
	import { afterUpdate } from 'svelte';
	// @ts-ignore
	import OtpItem from '$lib/components/ui/otp/OtpItem.svelte';
	export let numOfInputs = 6;
	export let value = '';
	export let separator = '';
	export let disableDefaultStyle = false;
	export let inputClass = '';
	export let wrapperClass = '';
	export let separatorClass = '';
	export let inputStyle = '';
	export let wrapperStyle = '';
	export let separatorStyle = '';
	export let numberOnly = false;
	export let placeholder = '';
	export let disabled = false;
	export let onlyShowMiddleSeparator = false;
	export let onSubmit: () => void = () => {};

	let codes = [
		...value.slice(0, numOfInputs).split(''),
		...Array(numOfInputs <= value.length ? 0 : numOfInputs - value.length).fill('')
	];
	let inputs = Array(numOfInputs).fill(null);
	afterUpdate(() => {
		codes = [
			...value.slice(0, numOfInputs).split(''),
			...Array(numOfInputs <= value.length ? 0 : numOfInputs - value.length).fill('')
		];
	});
	$: placeholders =
		placeholder.length < numOfInputs
			? [...placeholder.split(''), ...Array(numOfInputs - placeholder.length).fill('')]
			: placeholder.split('');
	// @ts-ignore
	$: value = codes.join('');
	$: if (value.length === 6) {
		onSubmit();
		inputs[0].focus();
	}
</script>

<div class={`${disableDefaultStyle ? '' : 'wrapper'} ${wrapperClass}`} style={wrapperStyle}>
	{#each codes as value, i (i)}
		<OtpItem
			num={numberOnly}
			bind:input={inputs[i]}
			bind:value
			index={i}
			bind:codes
			{inputs}
			nostyle={disableDefaultStyle}
			className={inputClass}
			style={inputStyle}
			placeholder={placeholders[i]}
			{disabled}
		/>
		{#if separator && i !== codes.length - 1 && (!onlyShowMiddleSeparator || (onlyShowMiddleSeparator && i === codes.length / 2 - 1 && numOfInputs % 2 === 0))}
			<span class={separatorClass} style={separatorStyle}>{separator}</span>
		{/if}
	{/each}
</div>

<style>
	.wrapper {
		display: flex;
		justify-content: space-evenly;
		align-items: center;
	}
</style>
