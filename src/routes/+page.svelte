<script>
  import BlinkerRow from "./BlinkerRow.svelte";
  import Title from "./Title.svelte";
  import { page } from "$app/stores";
  import TimelineMarker from "./TimelineMarker.svelte";
  import MarkerButtons from "./MarkerButtons.svelte";
  import { onMount } from "svelte";

  // get fps and interval from query params
  const fps = parseNumber(["fps"], 12, 0, 60);
  const interval = parseNumber(["interval", "intv"], 3, 0, fps);
  const noBlink = $page.url.searchParams.has("noblink") || $page.url.searchParams.has("no-blink");
  const speed = parseNumber(["speed", "spd"], 1, 0.01, 4, true);
  const maxLines = parseNumber(["maxlines", "max-lines", "maxline", "max-line"], 10, 1, 30);
  const showAllDelta =
    $page.url.searchParams.has("showalldelta") || $page.url.searchParams.has("show-all-delta");
  const hideDelta =
    $page.url.searchParams.has("hidedelta") || $page.url.searchParams.has("hide-delta");
  const noGrid = $page.url.searchParams.has("nogrid") || $page.url.searchParams.has("no-grid");
  /**
   * @param {string[]} aliases
   * @param {number} defaultValue
   * @param {number} min
   * @param {number} max
   * @returns {number}
   */
  function parseNumber(aliases, defaultValue, min, max, float = false) {
    for (const alias of aliases) {
      const value = $page.url.searchParams.get(alias);
      if (value == null) continue;
      const parsed = float ? parseFloat(value) : parseInt(value);
      return isNaN(parsed) ? 0 : Math.max(Math.min(parsed, max), min);
    }
    return defaultValue;
  }

  /** @type {BlinkerRow} */
  let blinkerRowComponent;
  /** @type {TimelineMarker} */
  let timelineMarkerComponent;

  // get correct vh
  let vh = 0;
  function updateVh() {
    vh = window.innerHeight;
  }
  onMount(() => {
    updateVh();
    window.addEventListener("resize", updateVh);
    return () => window.removeEventListener("resize", updateVh);
  });
</script>

<svelte:head>
  <meta name="description" content="A timing tool for animators." />
  <title>Animator Metronome</title>
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" />
  <link
    href="https://fonts.googleapis.com/css2?family=Roboto:wght@0,700&display=swap"
    rel="stylesheet"
  />
</svelte:head>

<div id="main" style={`height: ${vh}px`}>
  <Title {fps} {interval} {speed} />
  <BlinkerRow {fps} {interval} {noBlink} {speed} bind:this={blinkerRowComponent} />
  <TimelineMarker
    {fps}
    lineLimit={maxLines}
    {blinkerRowComponent}
    {showAllDelta}
    {hideDelta}
    {noGrid}
    bind:this={timelineMarkerComponent}
  />
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
    width: 100%;
    gap: 2.5vw;
    padding-top: 1vh;
  }
</style>
