<script>
  import { onMount } from "svelte";
  import FrameBlinker from "./FrameBlinker.svelte";
  export let fps = 12;
  export let interval = 3;
  export let speed = 1;
  export let noBlink = false;

  let cycle = 0;
  let current = 0;
  /** @type {number} */
  let intervalId = -1;
  function update() {
    current++;
    if (current >= fps) {
      cycle++;
      current = 0;
    }
  }
  onMount(() => {
    intervalId = setInterval(update, 1000 / speed / fps);
    return () => intervalId > 0 && clearInterval(intervalId);
  });

  export function getCurrent() {
    return current;
  }
  export function getCurrentCycle() {
    return cycle;
  }

  export function resetCycle() {
    current = 0;
    cycle = 0;
    clearInterval(intervalId);
    intervalId = setInterval(update, 1000 / speed / fps);
  }
</script>

<div id="blinkerRow">
  {#each Array.from({ length: fps }) as _, i}
    <FrameBlinker
      primary={i % interval === 0}
      isBlinking={!noBlink && i === current}
      index={i}
      {fps}
    />
  {/each}
</div>

<style>
  #blinkerRow {
    width: 92%;
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 1%;
  }
</style>
