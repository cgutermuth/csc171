<script>
	import * as d3 from 'd3';

	let { data, title, y, yLabel, color } = $props();

	const width = 440;
	const height = 230;
	const m = { top: 34, right: 34, bottom: 40, left: 56 };

	let xScale = $derived(d3.scaleLinear().domain([2020, 2024]).range([m.left, width - m.right]));
	let yScale = $derived(
		d3.scaleLinear().domain([0, d3.max(data, (d) => d[y])]).nice().range([height - m.bottom, m.top])
	);
	let line = $derived(
		d3
			.line()
			.x((d) => xScale(d.year))
			.y((d) => yScale(d[y]))
	);
	let series = $derived(
		Array.from(d3.group(data, (d) => d.state), ([state, rows]) => ({
			state,
			rank: rows[0].policy_rank,
			abbr: rows[0].state_abbr,
			rows: rows.slice().sort((a, b) => a.year - b.year)
		}))
	);

	let xAxis = $state();
	let yAxis = $state();
	$effect(() => {
		d3.select(xAxis).call(d3.axisBottom(xScale).ticks(5).tickFormat(d3.format('d')));
		d3.select(yAxis).call(d3.axisLeft(yScale).ticks(5));
	});
</script>

<svg viewBox="0 0 {width} {height}" role="img" aria-label={title}>
	<text x={m.left} y="18" font-size="15" font-weight="600">{title}</text>
	<g bind:this={xAxis} transform="translate(0, {height - m.bottom})" />
	<g bind:this={yAxis} transform="translate({m.left}, 0)" />
	<g fill="none" stroke-width="2.5" stroke-opacity="0.8">
		{#each series as s (s.state)}
			<path d={line(s.rows)} stroke={color(s.rank)}><title>{s.state}</title></path>
		{/each}
	</g>
	<g font-size="9">
		{#each series as s (s.state)}
			{@const last = s.rows[s.rows.length - 1]}
			<text x={xScale(last.year) + 4} y={yScale(last[y])} dy="0.32em">{s.abbr}</text>
		{/each}
	</g>
	<text x={(m.left + width - m.right) / 2} y={height - 8} font-size="12" text-anchor="middle">Year</text>
	<text transform="rotate(-90)" x={-(m.top + height - m.bottom) / 2} y="14" font-size="12" text-anchor="middle">{yLabel}</text>
</svg>
