<script>
	import { onMount } from "svelte";

	/** @type {boolean} */
	export let primary = true;
	export let isBlinking = false;
	export let index = 0;
	export let fps = 12;

	/** @type {HTMLDivElement} */
	let divBox;
	onMount(() => {
		setTimeout(() => {
			divBox.animate(
				[
					{ transform: "scale(0) rotate(180deg)" },
					{ transform: "scale(1) rotate(0deg)" },
				],
				{
					duration: 700,
					easing: "cubic-bezier(.6,1.67,.62,.84)",
					fill: "forwards",
				}
			);
		}, index * 1000 / fps);
	})
</script>

<div class={`frameBlinker${isBlinking ? ' blinking' : ''}${primary ? ' primary' : ''}`} >
	<div bind:this={divBox}></div>
</div>

<style>
	.frameBlinker {
		--non-blink: white;
		--blink-primary: #659cef;
		--blink-secondary: #EAEAEA;
	}

	.frameBlinker {
		width: 100%;
		max-width: 30vh;
	}

	.frameBlinker div {
		width: 100%;
		padding-bottom: 100%;
		border-radius: 15%;
		background-color: var(--non-blink);
		transition: background-color 0.5s;
		transform: scale(0) rotate(180deg);
	}

	.frameBlinker.blinking div {
		background-color: var(--blink-secondary);
		transition: background-color 0.05s;
	}

	.frameBlinker.primary.blinking div {
		background-color: var(--blink-primary);
	}
</style>
