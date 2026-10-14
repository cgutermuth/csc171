<script>
	import * as d3 from 'd3';
	import { onMount } from 'svelte';
	import Map from '#lib/Map.svelte';
	import Chain from '#lib/Chain.svelte';

	let data = $state([]);
	let state = $state(null); // hovered state name, owned by the page
	// Filtering removes other states' rows; the charts recompute their scales over the subset.
	let filtered = $derived(state === null ? data : data.filter((d) => d.state === state));

	onMount(async () => {
		data = await d3.csv('/state_data.csv', d3.autoType);
	});
</script>

<main>
	<h1>Reproductive health outcomes by state, 2020–2024</h1>
	{#if data.length}
		<Map {data} onHover={(s) => (state = s)} />
		<Chain data={filtered} />
	{:else}
		<p>Loading…</p>
	{/if}
</main>

<style>
	main {
		max-width: 1000px;
		margin: 0 auto;
		padding: 1rem;
		font-family: system-ui, sans-serif;
	}
</style>
