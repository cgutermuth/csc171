<script>
	import * as d3 from 'd3';
	import { onMount } from 'svelte';
	import Map from '#lib/Map.svelte';
	import Chain from '#lib/Chain.svelte';

	let data = $state([]);
	let state = $state(null); // hovered state name, owned by the page
	let hoveredRank = $state(null); // hovered policy rank (1-7), owned by the page
	const metrics = ['pct_zero_prenatal_visits', 'pct_maternal_morbidity', 'preterm_pct', 'infant_deaths'];
	// Hovering a state FILTERS to that state's rows. Hovering a rank AGGREGATES: one row per
	// year holding the mean across that rank's states (an average per state, not a sum).
	let filtered = $derived.by(() => {
		if (hoveredRank !== null) {
			const rows = data.filter((d) => d.policy_rank === hoveredRank);
			return Array.from(d3.group(rows, (d) => d.year), ([year, g]) => ({
				year,
				state: `Rank ${hoveredRank}`,
				policy_rank: hoveredRank,
				...Object.fromEntries(metrics.map((k) => [k, d3.mean(g, (d) => d[k])]))
			}));
		}
		return state === null ? data : data.filter((d) => d.state === state);
	});

	onMount(async () => {
		data = await d3.csv('/state_data.csv', d3.autoType);
	});
</script>

<main>
	<h1>Reproductive health outcomes by state, 2020–2024</h1>
	{#if data.length}
		<Map {data} onHover={(s) => (state = s)} onRankHover={(r) => (hoveredRank = r)} />
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
