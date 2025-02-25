<!--
Add the css snippet below to your global css file to do a 
full-screen + mobile friendly slider
	
html, body, main {
	height: 100%;
	overflow: hidden;
}

Usage:
<script>
	import Slider from "$components/helpers/Slider.svelte";
	import Slide from "$components/helpers/Slider.Slide.svelte";

	let sliderEl; // component binding

	sliderEl.next(); // navigation call
</script>

<Slider bind:this={sliderEl}>
	<Slide>
		<p>content</p>
	</Slide>
</Slider>
-->
<script>
	import PlayableText from "$components/PlayableText.svelte";

	import { setContext, onMount, tick } from "svelte";
	import { writable } from "svelte/store";

	export let direction = "horizontal";
	export let duration = "500ms";
	export let timing = "ease";

	export let count = 0;
	export let current = 0;

	export const next = () => move(1);
	export const prev = () => move(-1);
	export const jump = (val) => move(val, true);

	let children = 0;
	let index = 0;
	let width;
	let height;
	let isInView = false;
	let sliderEl;
	let translateEl;
	let root;
	let observer;

	let _direction = writable();
	let _width = writable();
	let _height = writable();
	let _current = writable();
	let _count = writable();

	const move = (val, jump) => {
		if (!isInView) return false;
		const target = jump ? val : index + val;
		index = Math.max(0, Math.min(children - 1, target));
		current = index;
	};

	const onIntersect = (e) => {
		isInView = e[0].isIntersecting;
	};

	$: w = direction === "horizontal" ? `${children * width}px` : "100%";
	$: h = direction === "vertical" ? `${children * height}px` : "100%";

	$: x = direction === "horizontal" ? `${index * width * -1}px` : 0;
	$: y = direction === "vertical" ? `${index * height * -1}px` : 0;

	$: sW = `width: ${w};`;
	$: sH = `height: ${h};`;
	$: sT = `transform: translate3d(${x}, ${y}, 0);`;
	$: sTD = `transition-duration: ${duration};`;
	$: sTTF = `transition-timing-function: ${timing};`;
	$: customStyle = `${sW} ${sH} ${sT} ${sTD} ${sTTF}`;

	// context
	$: _direction.set(direction);
	$: _width.set(width);
	$: _height.set(height);
	$: _current.set(current);
	$: context = {
		dir: _direction,
		cur: _current,
		w: _width,
		h: _height,
		count: _count
	};
	$: setContext("Slider", context);

	const playableText = () => {
		const playable = document.querySelectorAll("#phrase-by-phrase .playable");
		playable.forEach((el) => {
			const text = el.textContent;
			const id = el.dataset.id;
			const phraseI = el.dataset.phrase;
			el.textContent = "";
			new PlayableText({
				target: el,
				props: {
					id,
					phraseI,
					text
				}
			});
		});
	};

	onMount(async () => {
		await tick();
		playableText();

		children = translateEl.children.length;
		count = children;
		_count.set(count);
		observer = new IntersectionObserver(onIntersect, {
			root: null,
			rootMargin: "-1px"
		});
		observer.observe(sliderEl);
		height = height;
		width = width;
	});
</script>

<section
	aria-label="carousel"
	class="slider {direction}"
	bind:this={sliderEl}
	bind:clientWidth={width}
	bind:clientHeight={height}
>
	<div class="slides" bind:this={translateEl} style={customStyle}>
		<slot />
	</div>
</section>

<style>
	section {
		position: relative;
		height: calc(100% - 6rem);
		padding: 0;
		z-index: 2;
		pointer-events: none;
		width: calc(100% - 150px);
		margin: auto;
	}

	.slides {
		display: flex;
		flex-wrap: wrap;
		position: relative;
		width: 100%;
		height: 100%;
		transition-property: transform;
		z-index: 1;
	}

	.horizontal > .slides {
		flex-direction: row;
	}

	.vertical > .slides {
		flex-direction: column;
	}

	@media (max-width: 600px) {
		section {
			height: calc(100% - 4rem);
			width: 100%;
		}
	}
</style>
