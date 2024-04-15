<script lang="ts">
	import { version } from '$app/environment';
	import 'large-small-dynamic-viewport-units-polyfill';
	import svelteLogo from '$lib/images/svelte-logo.svg';
	import { Encoder, Byte } from '@nuintun/qrcode';
	import * as lz from 'lz-string';

	let inputValue: string = '';
	$: data = writeData(inputValue);
	const encoder = new Encoder({ version: 'Auto', level: 'L' });
	$: dataUrl = encoder.encode(new Byte(data)).toDataURL();
	$: {
		console.log(data);
		console.log(dataUrl);
	}
	$: typing = false;

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
		return lz.compressToEncodedURIComponent(str);
	}

	function decode(str: string): string {
		return lz.decompressFromEncodedURIComponent(str);
	}
</script>

<div class="main">
	<textarea
		bind:value={inputValue}
		on:focus={() => (typing = true)}
		on:blur={() => (typing = false)}
		maxlength={500}
		placeholder="500"
	></textarea>
	<img alt="qrcode" class="qrcode" width={177} height={177} src={typing ? svelteLogo : dataUrl} />
	<div>
		{version}
	</div>
</div>

<style lang="postcss">
	:global(body) {
		height: 100vh; /* For browsers that don't support CSS variables */
		height: calc(var(--1dvh, 1vh) * 100); /* This is the "polyfill" */
		height: 100dvh; /* This is for future browsers that support svh, dvh and lvh viewport units */
		@apply m-0 w-full p-0;
	}

	textarea {
		width: 100%;
		@apply grow border-2 border-black;
	}

	.qrcode {
		width: 177px;
		height: 177px;
	}

	.main {
		@apply flex h-full flex-col items-center justify-center  p-2;
	}
</style>
