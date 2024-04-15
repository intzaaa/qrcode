<script lang="ts">
	import { version } from '$app/environment';
	import 'large-small-dynamic-viewport-units-polyfill';
	import svelteLogo from '$lib/images/svelte-logo.svg';
	import { Encoder, Byte } from '@nuintun/qrcode';
	import * as lz from 'lz-string';
	import punycode from 'punycode';

	let inputValue: string = '';
	const encoder = new Encoder({ version: 'Auto', level: 'L' });
	$: typing = false;

	function readData() {
		const url = new URL(globalThis.window.location.href);
		const data = decode(url.searchParams.get('data') ?? '');
		inputValue = data;
		if (data !== '') {
			url.searchParams.delete('data');
			window.history.replaceState(null, '', url);
		}
	}

	readData();

	function writeData(data: string): string {
		const url = new URL(globalThis.window.location.href);
		url.searchParams.set('data', data);
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
	></textarea>
	<div class="qrcode-list">
		{#if !typing}
			{#each (inputValue?.match(/.{1,700}/gms) ?? []).map((v, i, a) => `<PART ${i + 1}/${a.length}>\n\n` + v) as partData, index}
				<div class="qrcode">
					<img
						alt="QR Code"
						title={`${index + 1}`}
						width={200}
						height={200}
						src={String(encoder.encode(new Byte(writeData(encode(partData)))).toDataURL())}
					/>
					<div>{index + 1}</div>
					<hidden>
						{(() => {
							console.log(partData);
						})()}
					</hidden>
				</div>
			{/each}
		{:else}
			<div style="padding-left: 25px">Make textarea lose focus to generate QR code</div>
		{/if}
	</div>
	<div>
		{version}
	</div>
</div>

<style lang="postcss">
	:global(*) {
		box-sizing: border-box;
	}
	:global(body) {
		height: 100vh; /* For browsers that don't support CSS variables */
		height: calc(var(--1dvh, 1vh) * 100); /* This is the "polyfill" */
		height: 100dvh; /* This is for future browsers that support svh, dvh and lvh viewport units */
		@apply m-0 w-full bg-gray-200 p-0;
	}

	textarea {
		width: 100%;
		@apply grow border-black p-2 font-serif text-sm outline-none;
	}

	img {
		@apply w-full grow;
	}

	hidden {
		display: none;
	}

	.qrcode {
		@apply flex h-auto w-[200px] shrink-0 flex-col items-center justify-center;
	}

	.qrcode-list {
		@apply flex h-fit min-h-[200px] w-full flex-row flex-nowrap items-center overflow-scroll bg-gray-100;
	}

	.main {
		@apply flex h-full flex-col items-center justify-center  p-2;
	}
</style>
