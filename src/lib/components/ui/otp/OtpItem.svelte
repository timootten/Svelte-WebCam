<script lang="ts">
	export let input: HTMLInputElement | null = null;
	export let index: number;
	export let codes: string[];
	export let inputs: HTMLInputElement[];
	export let nostyle: boolean;
	export let className: string;
	export let num: boolean;
	export let style: string;
	export let placeholder: string;
	export let disabled = false;

	let key: string;

	function shiftFocus(key2: string) {
		if (
			(!/[0-9]/.test(key2) && num && key2) ||
			['ArrowRight', 'ArrowLeft', 'Backspace'].includes(key2)
		)
			return;
		if (codes[index] === ' ') {
			codes[index] = '';
			return;
		}
		if (index !== inputs.length - 1) inputs[index + 1].focus();
	}

	function keyDownHandler(e: KeyboardEvent) {
		if (e.ctrlKey && e.key === 'z') {
			e.preventDefault();
		}
		key = e.key;
		if (codes[index].length >= 1 && !e.ctrlKey) shiftFocus(key);
	}

	function typeHandler(e: KeyboardEvent) {
		if (codes[index].length >= 1 || (!num && !/[0-9]/.test(e.key))) {
			e.preventDefault();
		}
	}

	function changeHandler(e: Event) {
		let val = (e.target as HTMLInputElement).value;
		if (/[0-9]/.test(val) || !num || !val) {
			codes = codes.map((c, i) => (i < index ? (c === '' ? ' ' : c) : i === index ? val[0] : c));
			if (!val) {
				let len = codes.length;
				let filtered = codes.filter((_, i) => i !== index);
				codes = [...filtered, ...Array(len - filtered.length).fill('')];
			}
			shiftFocus(key);
		}
	}

	function keyUpHandler(e: KeyboardEvent) {
		if ((e.key === 'Backspace' || e.key === 'ArrowLeft') && index !== 0) {
			inputs[index - 1]?.focus();
		} else if (e.key === 'ArrowRight' && index !== inputs.length - 1) {
			inputs[index + 1]?.focus();
		}
	}

	function pasteHandler(e: ClipboardEvent) {
		e.preventDefault();
		let paste = e.clipboardData?.getData('text');
		if (!paste) return;
		let pasteValue = paste.replace(num ? /[^0-9]/g : '', '').slice(0, codes.length - index);
		let newCodes = [
			...codes.slice(0, index),
			...pasteValue.split(''),
			...codes.slice(index + pasteValue.length)
		];
		codes = newCodes;
	}
</script>

<input
	class={`${nostyle ? '' : 'default-input'} ${className}`}
	bind:this={input}
	on:keydown={keyDownHandler}
	on:keyup={keyUpHandler}
	on:keypress={typeHandler}
	on:input={changeHandler}
	on:paste={pasteHandler}
	pattern="\d*"
	{style}
	value={codes[index]}
	{placeholder}
	{disabled}
/>

<style>
	.default-input {
		width: 30px;
		height: 40px;
		text-align: center;
		border: 1px solid black;
		margin: 0;
	}
</style>
