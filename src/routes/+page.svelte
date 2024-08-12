<script>
	import BlinkerRow from './BlinkerRow.svelte';
	import Title from './Title.svelte';
	import { page } from '$app/stores';
	import TimelineMarker from './TimelineMarker.svelte';
	import MarkerButtons from './MarkerButtons.svelte';

	// get fps and interval from query params
	const fps = parseNumber($page.url.searchParams.get('fps'), 12, 0, 60);
	const interval =
		$page.url.searchParams.get('interval') != null
			? parseNumber($page.url.searchParams.get('interval'), 3, 0, fps)
			: parseNumber($page.url.searchParams.get('intv'), 3, 0, fps);
	const noBlink = $page.url.searchParams.has('noblink');
	/**
	 * @param {string|null} value
	 * @param {number} defaultValue
	 * @param {number} min
	 * @param {number} max
	 * @returns {number}
	 */
	function parseNumber(value, defaultValue, min, max) {
		if (value === null) return defaultValue;
		const parsed = parseInt(value);
		return isNaN(parsed) ? 0 : Math.max(Math.min(parsed, max), min);
	}

	/** @type {BlinkerRow} */
	let blinkerRowComponent;
	/** @type {TimelineMarker} */
	let timelineMarkerComponent;
</script>

<svelte:head>
	<title>Animator Metronome</title>
	<link rel="preconnect" href="https://fonts.googleapis.com" />
	<link rel="preconnect" href="https://fonts.gstatic.com" />
	<link
		href="https://fonts.googleapis.com/css2?family=Roboto:ital,wght@0,700&display=swap"
		rel="stylesheet"
	/>
</svelte:head>

<div id="main">
	<Title {fps} {interval} />
	<BlinkerRow {fps} {interval} {noBlink} bind:this={blinkerRowComponent} />
	<TimelineMarker {blinkerRowComponent} bind:this={timelineMarkerComponent} />
	<MarkerButtons {timelineMarkerComponent} />
</div>

<style>
	* {
		--background-color: #d9dae1;
	}

	#main {
		background-color: var(--background-color);
		box-sizing: border-box;
		display: flex;
		flex-direction: column;
		justify-content: start;
		align-items: center;
		height: 100vh;
		width: 100%;
		gap: 2.5vw;
		padding-top: 1vh;
	}
</style>
