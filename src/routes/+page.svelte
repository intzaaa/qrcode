<script lang="ts">
	import 'large-small-dynamic-viewport-units-polyfill';
	import QrCode from 'svelte-qrcode';
	let area;
	let value;
</script>

<div class="main">
	<textarea bind:this={area} bind:value maxlength={1000} placeholder="maxlength=1000"></textarea>
	<QrCode {value} size={512} />
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
		height: 30vh;
		@apply border-black border-2;
	}
	:global(.qrcode) {
		@apply max-h-screen max-w-full;
	}
	.main {
		@apply p-0 m-2 flex flex-col items-center justify-center;
	}
</style>
