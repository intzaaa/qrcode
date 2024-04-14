<script lang="ts">
	import 'large-small-dynamic-viewport-units-polyfill';
	// @ts-ignore
	import QrCode from 'svelte-qrcode';
	let inputValue: string = '';
	$: data = writeData(inputValue);

	function readData() {
		const url = new URL(globalThis.window.location.href);
		inputValue = decode(url.searchParams.get('data') || '');
	}
	readData();

	function writeData(data: string): string {
		const url = new URL(globalThis.window.location.href);
		url.searchParams.set('data', encode(data || ''));
		return url.toString();
	}
	function encode(str: string): string {
		return encodeURIComponent(str);
	}
	function decode(str: string): string {
		return decodeURIComponent(str);
	}
</script>

<div class="main">
	<textarea bind:value={inputValue} maxlength={500} placeholder="500"></textarea>
	<QrCode value={data} size={512} />
</div>

<style lang="postcss">
	:global(body) {
		@apply m-0 w-full p-0;
	}
	textarea {
		width: 100%;
		height: 30vh;
		@apply border-2 border-black;
	}
	:global(.qrcode) {
		@apply max-h-screen max-w-full;
	}
	.main {
		@apply m-2 flex flex-col items-center justify-center p-0;
	}
</style>
