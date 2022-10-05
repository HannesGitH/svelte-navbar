<script lang="ts">
	// import Button, { Label } from '@smui/button';
    import { createEventDispatcher } from 'svelte';

	const dispatch = createEventDispatcher();

	export const isCurrentlyActivce : boolean = false;
	export const color : string = 'white';

	function onClick() {
		dispatch('click', {
			text: 'button clicked'
		});
	}
</script>
<div id="defs" style="--color: {color}"></div>

<button class=" mdc-button mdc-button__ripple mdc-button--unelevated navElem {isCurrentlyActivce ? 'active' : ''}" on:click={onClick}><slot></slot></button>

<style lang="scss">
	@use '@material/button/index' as mdc-button;
	@use 'sass:color';
	$secondary: var(--color);

	:global(.navElem) {
		@include mdc-button.container-fill-color($secondary);
		// @include mdc-button.ink-color($secondary);
		@include mdc-button.shape-radius(999px);
		@include mdc-button.outline-width(0, 1rem);
		// @include mdc-button.height(1rem);
		font-style: italic;

		z-index: 3;

		.active {
			scale: 1.2;
		}

		&:hover {
			opacity: 0.9;
		}
		&:active {
			opacity: 0.5;
		}
	}
</style>
