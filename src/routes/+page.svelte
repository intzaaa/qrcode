<script lang="ts">
	import { version } from '$app/environment';
	import { page } from '$app/stores';
	import { pushState, replaceState } from '$app/navigation';
	import 'large-small-dynamic-viewport-units-polyfill';
	import { Encoder, Numeric, Byte } from '@nuintun/qrcode';
	import * as lz from 'lz-string';
	import GraphemeSplitter from 'grapheme-splitter';
	import { onMount } from 'svelte';

	let inputValue: string;
	const splitter = new GraphemeSplitter();
	const QREncoder = new Encoder({ version: 'Auto', level: 'L' });
	$: typing = false;
	$: mode = 'URL';

	function readData() {
		const url = new URL(globalThis.window.location.href);
		const data = dataDecode(url.searchParams.get('data') ?? '');
		inputValue = data;
		if (data !== '') {
			url.searchParams.delete('data');
			pushState(url, {});
		}
	}

	onMount(() => {
		readData();
	});

	function writeData(data: string): string {
		const url = new URL(globalThis.window.location.href);
		url.searchParams.set('data', data);
		return url.toString();
	}

	function dataEncode(str: string): string {
		return lz.compressToEncodedURIComponent(str);
	}

	function dataDecode(str: string): string {
		return lz.decompressFromEncodedURIComponent(str);
	}

	// 每 2000 个字符分割一次
	function splitString(str: string, max: number): Array<string> {
		const strArray = splitter.splitGraphemes(str);
		const length = strArray.length;
		const result = new Array<string>();
		let temp = '';
		strArray.forEach((v, i) => {
			if (temp.length + v.length >= max || i === length - 1) {
				result.push(temp);
				temp = '';
			}
			temp += v;
		});
		return result;
	}

	const modes = ['URL', 'Raw'];
</script>

<div class="main">
	<textarea
		bind:value={inputValue}
		on:focus={() => (typing = true)}
		on:blur={() => (typing = false)}
	></textarea>
	<div class="qrcode-list">
		{#if !typing}
			{#each splitString(inputValue ?? '', 200) as string, index}
				{#key mode}
					<div class="qrcode">
						{#if mode === 'URL'}
							<img
								alt="QR Code"
								title={`${index + 1}`}
								width={200}
								height={200}
								src={QREncoder.encode(new Byte(writeData(dataEncode(string)))).toDataURL()}
							/>
						{:else if mode === 'Raw'}
							<img
								alt="QR Code"
								title={`${index + 1}`}
								width={200}
								height={200}
								src={QREncoder.encode(new Byte(string)).toDataURL()}
							/>
						{/if}
						<div>{index + 1}</div>
						<hidden>
							{(() => {
								console.log(string);
								console.log(dataEncode(string));
							})()}
						</hidden>
					</div>
				{/key}
			{/each}
		{:else}
			<div style="padding-left: 25px">Make textarea lose focus to generate QR code</div>
		{/if}
	</div>
	<div class="footer">
		<div>
			<select bind:value={mode}>
				<summary>Mode</summary>
				{#each modes as mode}
					<option value={mode}>{mode}</option>
				{/each}
			</select>
		</div>
		<div>{version}</div>
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
		@apply m-0 w-full bg-gray-200 p-2;
	}

	textarea {
		width: 100%;
		@apply grow border-black p-2 font-serif text-base outline-none;
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
		@apply flex h-full flex-col items-center justify-center rounded-xl;
	}

	.footer {
		@apply flex h-auto w-full flex-row items-center justify-between p-2 text-xs;
	}
</style>
