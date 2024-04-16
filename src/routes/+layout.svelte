<script lang="ts">
	import '../app.pcss';
	import { dev } from '$app/environment';
	navigator.serviceWorker.register('/service-worker.js', {
		type: dev ? 'module' : 'classic'
	});
</script>

<svelte:head>
	<title>QR Code</title>
</svelte:head>

<slot></slot>
