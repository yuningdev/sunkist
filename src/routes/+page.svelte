<script>
	// @ts-nocheck

	import { default as SankeyChart } from '$lib/components/Sankey.svelte';
	import { onMount } from 'svelte';

	let deltaY = undefined;
</script>

<svelte:head>
	<title>Home</title>
	<meta name="description" content="Svelte demo app" />
</svelte:head>

<div id="app">
	<div>
		<div class="toolbar">
			<div>101 Taiwanese Abroad</div>
		</div>

		<div class="toolbar-filter">
			<div>Uploader</div>
			<div>selector</div>
		</div>
	</div>

	<SankeyChart />
</div>

<style lang="scss">
	#app {
		width: 100vw;
		height: 100vh;
		overflow: hidden;
	}
	#sankey-canvas {
		width: 100%;
		height: 100%;
		overflow: hidden;
	}
	.toolbar {
		color: white;
		background: #40679e;
		width: 100%;
		height: 24px;
	}
	.toolbar-filter {
		display: flex;
		flex-direction: row;
		gap: 12px;
		border: 1px solid #f3f3f3;
		width: 100%;
		height: 72px;
	}
	.content {
		transition: transform 1s ease;
		width: 100%;
		height: 100%;
	}
</style>
