<!--
  @component
  Generates an SVG y-axis. This component is also configured to detect if your y-scale is an ordinal scale. If so, it will place the markers in the middle of the bandwidth.
 -->
<script>
	import { getContext } from "svelte";

	const { data, y, padding, xRange, yScale, yGet } = getContext("LayerCake");

	export let animate = false;
	export let active;

	/** @type {Boolean} [gridlines=true] - Extend lines from the ticks into the chart space */
	export let gridlines = true;

	/** @type {Boolean} [tickMarks=false] - Show a vertical mark for each tick. */
	export let tickMarks = false;

	/** @type {Function} [formatTick=d => d] - A function that passes the current tick value and expects a nicely formatted value in return. */
	export let formatTick = (d) => d;

	/** @type {Number|Array|Function} [ticks=4] - If this is a number, it passes that along to the [d3Scale.ticks](https://github.com/d3/d3-scale) function. If this is an array, hardcodes the ticks to those values. If it's a function, passes along the default tick values and expects an array of tick values in return. */
	export let ticks = 4;

	/** @type {Number} [xTick=0] - How far over to position the text marker. */
	export let xTick = 0;

	/** @type {Number} [yTick=0] - How far up and down to position the text marker. */
	export let yTick = 0;

	/** @type {Number} [dxTick=0] - Any optional value passed to the `dx` attribute on the text marker and tick mark (if visible). This is ignored on the text marker if your scale is ordinal. */
	export let dxTick = 0;

	/** @type {Number} [dyTick=-4] - Any optional value passed to the `dy` attribute on the text marker and tick mark (if visible). This is ignored on the text marker if your scale is ordinal. */
	export let dyTick = -4;

	/** @type {String} [textAnchor='start'] The CSS `text-anchor` passed to the label. This is automatically set to "end" if the scale has a bandwidth method, like in ordinal scales. */
	export let textAnchor = "start";

	$: isBandwidth = typeof $yScale.bandwidth === "function";

	$: tickVals = Array.isArray(ticks)
		? ticks
		: isBandwidth
		? $yScale.domain()
		: typeof ticks === "function"
		? ticks($yScale.ticks())
		: $yScale.ticks(ticks);
</script>

<g class="axis y-axis" transform="translate({-$padding.left}, 0)">
	{#if isBandwidth}
		{#each $data as d, i ($y(d))}
			<g
				class="tick tick-{$y(d)}"
				class:animate={animate && active}
				transform="translate({$xRange[0] +
					(isBandwidth ? $padding.left : 0)}, {$yGet(d)})"
			>
				{#if gridlines !== false}
					<line
						class="gridline"
						x2="100%"
						y1={isBandwidth ? $yScale.bandwidth() / 2 : 0}
						y2={isBandwidth ? $yScale.bandwidth() / 2 : 0}
					></line>
				{/if}
				{#if tickMarks === true}
					<line
						class="tick-mark"
						x1="0"
						x2={isBandwidth ? -6 : 6}
						y1={isBandwidth ? $yScale.bandwidth() / 2 : 0}
						y2={isBandwidth ? $yScale.bandwidth() / 2 : 0}
					></line>
				{/if}
				<text
					class:bandwidth={isBandwidth}
					class:small={isBandwidth && tickVals.length > 8}
					x={xTick}
					y={isBandwidth ? $yScale.bandwidth() / 2 + yTick : yTick}
					dx={isBandwidth ? -9 : dxTick}
					dy={isBandwidth ? 4 : dyTick}
					style="text-anchor:{isBandwidth ? 'end' : textAnchor};"
					>{formatTick($y(d))}</text
				>
			</g>
		{/each}
	{:else}
		{#each tickVals as tick (tick)}
			<g
				class="tick tick-{tick}"
				class:animate={animate && active}
				transform="translate({$xRange[0] +
					(isBandwidth ? $padding.left : 0)}, {$yScale(tick)})"
			>
				{#if gridlines !== false}
					<line
						class="gridline"
						x2="100%"
						y1={isBandwidth ? $yScale.bandwidth() / 2 : 0}
						y2={isBandwidth ? $yScale.bandwidth() / 2 : 0}
					></line>
				{/if}
				{#if tickMarks === true}
					<line
						class="tick-mark"
						x1="0"
						x2={isBandwidth ? -6 : 6}
						y1={isBandwidth ? $yScale.bandwidth() / 2 : 0}
						y2={isBandwidth ? $yScale.bandwidth() / 2 : 0}
					></line>
				{/if}
				<text
					class:bandwidth={isBandwidth}
					class:small={isBandwidth && tickVals.length > 8}
					x={xTick}
					y={isBandwidth ? $yScale.bandwidth() / 2 + yTick : yTick}
					dx={isBandwidth ? -9 : dxTick}
					dy={isBandwidth ? 4 : dyTick}
					style="text-anchor:{isBandwidth ? 'end' : textAnchor};"
					>{formatTick(tick)}</text
				>
			</g>
		{/each}
	{/if}
</g>

<style>
	.tick {
		font-size: 0.725em;
		font-weight: 200;
	}

	.tick line {
		stroke: var(--color-fg);
	}
	.tick .gridline {
		stroke-dasharray: 2;
	}

	.tick text {
		font-family: var(--sans);
		fill: var(--color-grey-blue);
		text-transform: capitalize;
	}
	.tick text.bandwidth {
		font-family: "Newsagent";
		font-variant-alternates: swash(fancy);
		font-size: 1.8rem;
		fill: var(--color-fg);
	}
	.tick text.small {
		font-size: 1rem;
		font-family: var(--sans);
		font-variant-alternates: normal;
	}

	.tick.tick-0 line {
		stroke-dasharray: 0;
	}

	g.animate {
		transition: none;
	}
	g.animate.animate {
		transition: transform var(--1s) var(--1s);
	}

	@media (max-width: 600px) {
		.tick text.bandwidth {
			font-size: 1rem;
		}
		.tick text.small {
			font-size: var(--12px);
		}
	}
</style>
