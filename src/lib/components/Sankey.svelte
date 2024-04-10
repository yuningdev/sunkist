<!-- D3 Sankey chart using circle node instead of rectangle node -->
<!-- https://stackoverflow.com/questions/23208119/d3-sankey-chart-using-circle-node-instead-of-rectangle-node -->
<script>
	// @ts-nocheck

	import * as d3 from 'd3';
	import { sankey as d3Sankey, sankeyCenter, sankeyRight, sankeyLinkHorizontal } from 'd3-sankey';
	import { onDestroy, onMount } from 'svelte';
	// ref https://observablehq.com/d/c3bc50622bf02356
	const dimensions = { width: 1200, height: 1600, padding: 60, margins: 60 };
	const nodeColors = {
		0: '#FF407D',
		1: '#FFCAD4',
		2: '#40679E'
	};

	const lineColors = {
		0: '#FF407D',
		1: '#FFCAD4',
		2: '#40679E'
	};

	const lineWidthScale = 2;

	// the right shift of labels show on nodes
	const labelShiftX = 12;

	const charId = 'sankey_01';

	let rootDom;

	let zoom;

	const handleZoom = (event) => {
		d3.select(`#${charId} svg g`).attr('transform', event.transform);
	};

	/**
	 * @type {any}
	 */
	let data;

	/**
	 * @param {string} source
	 */
	const parseGraph = async (source) => {
		const links = await d3.csv(source, ({ source, target, value }) => {
			return {
				source,
				target,
				value: !value || isNaN((value = +value)) ? 1 : value
				// color: "red",
			};
		});

		const nodeByName = new Map();
		for (const link of links) {
			if (!nodeByName.has(link.source)) nodeByName.set(link.source, { name: link.source });
			if (!nodeByName.has(link.target)) nodeByName.set(link.target, { name: link.target });
		}

		return { nodes: Array.from(nodeByName.values()), links };
	};

	const run = async (data) => {
		// const data = await parseGraph('111_study_abroad.csv');
		const sankey = d3Sankey();
		sankey.nodeId((d) => d.name);
		sankey.nodeAlign(sankeyRight);
		sankey.nodeSort(null);
		sankey.nodeWidth(8);
		sankey.nodePadding(dimensions.padding);
		sankey.extent([
			[0, 5],
			[dimensions.width, dimensions.height]
		]);

		const sankeyData = sankey(data);

		rootDom = d3.select(`#${charId}`);
		const svg = rootDom
			.append('svg')
			.attr('width', dimensions.width)
			.attr('height', dimensions.height)
			.attr('overflow', 'visible');

		const chart = svg
			.append('g')
			.attr('transform', `translate(${dimensions.margins},${dimensions.margins})`)
			.attr('height', dimensions.height - dimensions.margins * 2)
			.attr('width', dimensions.width - dimensions.margins * 2)
			.attr('overflow', 'visible');

		// const adjustor = (i) => {
		// 	if (i === 8) {
		// 		return 30;
		// 	} else if (i === 6) {
		// 		return -30;
		// 	} else return 0;
		// };

		// Draw title
		// chart
		// 	.append('text')
		// 	.text('101 Taiwanese Abroad')
		// 	.attr('dominant-baseline', 'middle')
		// 	.attr('font-size', '31.25')
		// 	.attr('font-weight', '600')
		// 	.attr('font-family', 'arial');

		// This is Rect Node
		// const nodes = chart
		// 	.append('g')
		// 	.selectAll('rect')
		// 	.data(sankeyData.nodes)
		// 	.join('rect')
		// 	.attr('x', (d) => d.x0)
		// 	.attr('y', (d) => d.y0)
		// 	.attr('fill', (d) => {
		// 		return nodeColors[d.layer];
		// 	})
		// 	.attr('height', (d) => d.y1 - d.y0)
		// 	.attr('width', (d) => d.x1 - d.x0);

		const nodes = chart
			.append('g')
			.selectAll('circle')
			.data(sankeyData.nodes)
			.join('circle')
			.attr('cx', (d) => d.x0 + (d.x1 - d.x0) / 2)
			.attr('cy', (d) => d.y0 + (d.y1 - d.y0) / 2)
			.attr('r', (d) => ((d.y1 - d.y0) / 2 / 2))
			.attr('fill', (d) => "#f3f3f3");
		// .attr('fill', (d) => {
		// 	return nodeColors[d.layer];
		// })
		// .attr('height', (d) => d.y1 - d.y0)
		// .attr('width', (d) => d.x1 - d.x0);

		console.log('sankeyData.links', sankeyData.links);
		const links = chart
			.append('g')
			.attr('fill', 'none')
			.attr('stroke-opacity', 0.2)
			.selectAll('path')
			.data(sankeyData.links)
			.join('path')
			.attr('d', sankeyLinkHorizontal())
			.attr('stroke', (d) => {
				return lineColors[d.source.layer];
			})
			.attr('stroke-width', (d) => {
				// return Math.max(1, Math.sqrt(d.width * lineWidthScale))
				// return d.width * lineWidthScale;
				return  Math.max(lineWidthScale, Math.sqrt(d.width * lineWidthScale))
			})
			.on('mouseover', (event, eve) => {
				const target = d3.select(event.currentTarget);
				return target.attr('stroke-opacity', 0.8);
			})
			.on('mouseout', (event, eve) => {
				const target = d3.select(event.currentTarget);
				return target.attr('stroke-opacity', 0.2);
			});

		const labelNames = chart
			.append('g')
			.selectAll('text')
			.data(sankeyData.nodes)
			.join('text')
			.text((d) => d.name)
			.attr('class', (d) => d.depth)
			.attr('x', (d) => d3.mean([d.x0, d.x1]) + labelShiftX)
			.attr('y', (d) => d3.mean([d.y0, d.y1]))
			// .attr('dy', (d) => `${-5 + adjustor(d.index)}px`)
			// .attr('text-anchor', 'middle')
			.attr('dominant-baseline', 'middle')
			.attr('font-family', 'arial')
			.attr('font-weight', '400')
			.attr('font-size', '14');

		const labelValues = chart
			.append('g')
			.selectAll('text')
			.data(sankeyData.nodes)
			.join('text')
			.text((d) => `${d3.format('~s')(d.value)}`)
			.attr('x', (d) => d3.mean([d.x0, d.x1]) + labelShiftX)
			.attr('y', (d) => d3.mean([d.y0, d.y1]))
			.attr('dy', (d) => 15)
			// .attr('text-anchor', 'middle')
			.attr('dominant-baseline', 'middle')
			.attr('font-family', 'arial')
			.attr('font-weight', '200')
			.attr('font-size', '14');

		svg.node();
	};

	const initZoom = (rootDom) => {
		if (rootDom) {
			zoom = d3.zoom().on('zoom', handleZoom);
			rootDom.call(zoom);
		}
	};

	onMount(async () => {
		data = await parseGraph('111_study_abroad.csv');
	});

	onDestroy(() => {
		if (rootDom) {
			rootDom.selectAll('*').remove();
		}
	});

	$: if (data) run(data);

	$: initZoom(rootDom);
</script>

<div id={charId} class="wrapper" />

<style lang="scss">
	.wrapper {
		overflow: hidden;
	}
</style>
