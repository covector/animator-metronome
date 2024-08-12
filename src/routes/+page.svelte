<script>
	import BlinkerRow from './BlinkerRow.svelte';
	import Title from './Title.svelte';
	import { page } from '$app/stores';
	import TimelineMarker from './TimelineMarker.svelte';

	// get fps and interval from query params
	const fps = parseNumber($page.url.searchParams.get('fps'), 12);
	const interval =
		$page.url.searchParams.get('interval') != null
			? parseNumber($page.url.searchParams.get('interval'), 3)
			: parseNumber($page.url.searchParams.get('intv'), 3);
	/**
	 * @param {string|null} value
	 * @param {number} defaultValue
	 * @returns {number}
	 */
	function parseNumber(value, defaultValue) {
		if (value === null) return defaultValue;
		const parsed = parseInt(value);
		return isNaN(parsed) ? 0 : parsed;
	}

	/** @type {BlinkerRow} */
	let blinkerRowComponent;
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
	<BlinkerRow {fps} {interval} bind:this={blinkerRowComponent} />
	<TimelineMarker {blinkerRowComponent} />
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
