<script lang="ts">
	import { onMount } from 'svelte';
	import { interpolateSequence } from 'flubber2';
	const svgSize = 1024;
	//this depends on the actual svg used for the bottom bar, it has a viewBoxHeight of 1024 and the actual content starts at 920
	const svgContentStart = { whenBottom: 920, whenleft: 125 };

	export let color = '#a7c347';
	export let marginBetweenIconsInPercent = 2;
	export let rowSizeInPx: number = 70;
	let paths = {
		left: 'M-107.884 1120.17L-105.799-47.8354C-105.701-102.542-61.2735-146.811-6.56687-146.714L17.4523-146.671C72.159-146.573 116.428-102.145 116.331-47.4388L114.245 1120.57C114.148 1175.27 69.7198 1219.54 15.0132 1219.45L-9.00604 1219.4C-63.7127 1219.3-107.982 1174.88-107.884 1120.17Z',
		bottomToLeft:
			'M-6.5625-146.719C-61.2692-146.816-105.715-102.55-105.812-47.8438C-105.812-47.8438-191.522 849.744-84.6475 1051.67C83.3629 1252.74 1090.94 1133.62 1090.94 1133.62C1145.64 1133.83 1190.17 1089.64 1190.38 1034.94L1190.44 1010.91C1190.64 956.2 1146.49 911.705 1091.78 911.5L581.081 909.589C323.465 908.625 115.001 699.003 115.464 441.385L116.344-47.4375C116.441-102.144 72.1442-146.559 17.4375-146.656L-6.5625-146.719Z',
		bottom:
			'M-73.6978 920.009L1094.31 920.009C1149.02 920.009 1193.37 964.357 1193.37 1019.06L1193.37 1043.08C1193.37 1097.79 1149.02 1142.14 1094.31 1142.14L-73.6978 1142.14C-128.405 1142.14-172.753 1097.79-172.753 1043.08L-172.753 1019.06C-172.753 964.357-128.405 920.009-73.6978 920.009Z'
	};
	export let scrollProgress = 0;
	let interpolator = (p: number) => paths.bottom;
	onMount(async () => {
		const pathls = [paths.bottom, paths.bottomToLeft, paths.left];
		interpolator = (await interpolateSequence(pathls, { loop: false })) as (p: number) => string;
		updateSlot();
	});
	const scaledToExponents = (x: number, l: number) => {
		const val = x - l / 2;
		if (val < 0) return 1 / (1 - val);
		else if (val == 0) return 1;
		else return val + 1;
	};
	$: elemtsrowStyle =
		scrollProgress <= 0
			? `height:${rowSizeInPx}px`
			: scrollProgress >= 1
			? `width:${rowSizeInPx}px`
			: 'overflow:hidden';
	let slotti: Element | null = null;

	const updateSlot = ()=>{
		if (slotti) {
			const collection = slotti.children;
			const rect = slotti.getBoundingClientRect();
			const clength = collection.length;
			const calcXY = (index: number, progress: number, x?: number, y?: number) => {
				const scaledProgress = (progress ** scaledToExponents(index + 1.5, clength) * Math.PI) / 2;
				const modifiedX = x ? x / rect.width : (index + 0.5) / (clength + 1);
				const modifiedY = y ? y / rect.height : (clength - index) / (clength + 1);
				return {
					x: (1 - Math.sin(scaledProgress)) * modifiedX * rect.width,
					y: (1 - Math.cos(scaledProgress)) * modifiedY * rect.height,
					// + (svgSize - svgContentStart.whenBottom) / 2,
					rad: scaledProgress
				};
			};
			let widthAccu = 50;
			let heightAccu = rect.height - 100;
			for (let i = 0; i < collection.length; i++) {
				const element = collection[i];
				if (element instanceof HTMLElement) {
					const elemRect = element.getBoundingClientRect();
					const { x, y, rad } = calcXY(i, scrollProgress, widthAccu, heightAccu);
					widthAccu += elemRect.width + (marginBetweenIconsInPercent * rect.width) / 100;
					heightAccu -= elemRect.height + (marginBetweenIconsInPercent * rect.height) / 100;
					element.style.position = 'absolute';
					element.style.transformOrigin = 'bottom left';
					element.style.bottom = `${y + 10}px`;
					element.style.left = `${x + 10}px`;
					element.style.transform = `rotateZ(${rad}rad)`;
				}
			}
		}
	}

	$: {
		slotti,scrollProgress ,updateSlot();
	}
	let window: any;
</script>

<div id="elemts-row" bind:this={slotti} style={elemtsrowStyle}>
	<slot>
		<p>Lorem ipsum dolor sit amet.</p>
	</slot>
</div>
<div
	id="wrapper"
	style="
	--color: {color};
	--progress: {scrollProgress};
	width: calc({((scrollProgress ** 7 * svgSize) / svgContentStart.whenleft) * rowSizeInPx}px + {(1 -
		scrollProgress ** 7) *
		100}%); 
	height: calc({(((1 - scrollProgress ** (1 / 7)) * svgSize) /
		(svgSize - svgContentStart.whenBottom)) *
		rowSizeInPx}px + {scrollProgress ** (1 / 7) * 100}%);
	"
>
	<svg
		height="100%"
		stroke-miterlimit="10"
		style="fill-rule:nonzero;clip-rule:evenodd;stroke-linecap:round;stroke-linejoin:round;"
		version="1.1"
		viewBox="0 0 {svgSize} {svgSize}"
		preserveAspectRatio="none"
		pointer-events="none"
		width="100%"
		xmlns="http://www.w3.org/2000/svg"
		xmlns:xlink="http://www.w3.org/1999/xlink"
	>
		<defs />
		<clipPath id="ArtboardFrame">
			<rect height="100%" width="100%" x="0" y="0" />
		</clipPath>
		<g clip-path="url(#ArtboardFrame)" id="Layer-1">
			<path
				id="main"
				d={interpolator(scrollProgress)}
				fill-rule="nonzero"
				opacity="1"
				stroke="none"
			/>
		</g>
	</svg>
</div>

<!--? ich weiss nicht mehr wof??r der genau n??tig war, aber er hat was sinnvolles gemacht, aber gleichzeitig leider auch enormen CLS verursacht (was ewig herauszufinden gedauert hat) -->
<!-- <svelte:window bind:scrollY={window} bind:innerWidth={window} /> -->

<style lang="scss">
	#wrapper {
		@include full;
		pointer-events: none;
		overflow: visible;
		// opacity: 0.5;
		z-index: 0;
	}

	#elemts-row {
		@include full;
		z-index: 1;
		width: 100%;
		overflow: auto;

		white-space: nowrap;

		-ms-overflow-style: none; /* IE and Edge */
		scrollbar-width: none; /* Firefox */
		&::-webkit-scrollbar {
			display: none;
		}
		& + * {
			// position: absolute;
		}

		& * {
			transition: scale 0.5s ease;
		} 
	}
	#main {
		// @include smui;
		fill: var(--color);
	}
</style>
