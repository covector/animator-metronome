<script>
	import { onMount } from "svelte";
    import FrameBlinker from "./FrameBlinker.svelte";
    export let fps = 12;
    export let interval = 3;

    let cycle = 0;
    let current = 0;
    /** @type {number} */
    let intervalId = -1;
    onMount(() => {
        intervalId = setInterval(() => {
            current++;
            if (current >= fps) {
                cycle++;
                current = 0;
            }
        }, 1000 / fps);
        return () => intervalId > 0 && clearInterval(intervalId);
    });

    export function getCurrent() {
        return current;
    }
    export function getCurrentCycle() {
        return cycle;
    }
</script>

<div id="blinkerRow">
    {#each Array.from({ length: fps }) as _, i}
        <FrameBlinker primary={i % interval === 0} isBlinking={i === current} />
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