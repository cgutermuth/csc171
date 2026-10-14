<script>
	import * as d3 from 'd3';
	import { feature } from 'topojson-client';
	import topology from 'us-atlas/states-10m.json';

	let { data, onHover } = $props();

	const width = 960;
	const height = 600;
	const states = feature(topology, topology.objects.states).features;
	const projection = d3.geoAlbersUsa().fitSize([width, height - 60], { type: 'FeatureCollection', features: states });
	const path = d3.geoPath(projection);

	const ranks = d3.range(1, 8);
	const color = d3.scaleOrdinal().domain(ranks).range(d3.schemeRdYlBu[7]);

	// policy_rank is constant per state, so the first row per state is enough
	let rankByState = $derived(new Map(data.map((d) => [d.state, d.policy_rank])));
	let tiles = $derived(
		states.map((s) => ({ name: s.properties.name, d: path(s), rank: rankByState.get(s.properties.name) }))
	);
</script>

<svg viewBox="0 0 {width} {height}" role="img" aria-label="Map of US states colored by abortion policy rank">
	<g>
		{#each tiles as t (t.name)}
			{#if t.d}
				<path d={t.d} fill={t.rank ? color(t.rank) : '#ddd'} stroke="#fff" stroke-width="0.75" onmouseenter={() => onHover(t.name)} onmouseleave={() => onHover(null)}>
					<title>{t.name}: policy rank {t.rank}</title>
				</path>
			{/if}
		{/each}
	</g>
	<g transform="translate(250, {height - 40})">
		<text y="-8" font-size="13">Policy rank (1 = most restrictive, 7 = most protective)</text>
		{#each ranks as r, i (r)}
			<rect x={i * 60} width="60" height="14" fill={color(r)} />
			<text x={i * 60 + 30} y="30" font-size="12" text-anchor="middle">{r}</text>
		{/each}
	</g>
</svg>

<style>
	svg {
		width: 100%;
		max-width: 960px;
		height: auto;
	}
</style>
