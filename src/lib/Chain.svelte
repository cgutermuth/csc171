<script>
	import * as d3 from 'd3';
	import LineChart from './LineChart.svelte';

	let { data } = $props();

	const color = d3.scaleOrdinal().domain(d3.range(1, 8)).range(d3.schemeRdYlBu[7]);
	const charts = [
		{ title: 'Prenatal Care', y: 'pct_zero_prenatal_visits', yLabel: '% of births with no prenatal visits' },
		{ title: 'Maternal Complications', y: 'pct_maternal_morbidity', yLabel: '% of births with maternal morbidity' },
		{ title: 'Preterm Births', y: 'preterm_pct', yLabel: '% of preterm births' },
		{ title: 'Infant Mortality', y: 'infant_deaths', yLabel: 'Number of infant deaths' }
	];
</script>

<section>
	<h2>Where does the reproductive health chain break first after access is lost?</h2>
	<div class="grid">
		{#each charts as c (c.y)}
			<LineChart {data} title={c.title} y={c.y} yLabel={c.yLabel} {color} />
		{/each}
	</div>
	<p class="note">Each line is one state, colored by policy rank (red = most restrictive, blue = most protective).</p>
</section>

<style>
	.grid {
		display: flex;
		flex-direction: column;
		gap: 0.5rem;
	}
	h2 {
		margin-top: 0;
		font-size: 1.2rem;
	}
	.note {
		font-size: 0.85rem;
		color: #555;
	}
</style>
