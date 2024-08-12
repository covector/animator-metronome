<script>
	import { onMount } from "svelte";
    import BlinkerRow from "./BlinkerRow.svelte";

    /** @type {BlinkerRow} */
    export let blinkerRowComponent;

    // marking
    /** @type {number[][]} */
    let markers = [];
    let startCycle = 0;
    function mark() {
        if (markers.length === 0) {
            startCycle = blinkerRowComponent.getCurrentCycle();
        }
        const lineNumber = blinkerRowComponent.getCurrentCycle() - startCycle;
        if (lineNumber >= markers.length) {
            for (let i = markers.length; i < lineNumber + 1; i++) {
                markers.push([]);
            }
        }
        if (markers[lineNumber].includes(blinkerRowComponent.getCurrent())) return;
        markers[lineNumber].push(blinkerRowComponent.getCurrent());
        markers = markers;
        updateLineYs();
    }
    const clearMarkersDelay = 0.5; // seconds
    function clearMarkers() {
        // animate lines
        Array.from(timerlineMarkerComponent.getElementsByClassName("lines")).forEach(l => {
            l.animate([
                { strokeDashoffset: 0 },
                { strokeDashoffset: width }
            ], {
                duration: clearMarkersDelay * 1000,
                easing: "cubic-bezier(.59,.34,.33,.94)",
                fill: "forwards"
            });
        });
        // animate markers
        Array.from(timerlineMarkerComponent.getElementsByClassName("markers")).forEach(m => {
            m.animate([
                { transform: "scale(1) rotate(45deg)" },
                { transform: "scale(0) rotate(-45deg)" }
            ], {
                duration: clearMarkersDelay * 1000,
                easing: "cubic-bezier(.59,.34,.33,.94)",
                fill: "forwards"
            });
        });
        setTimeout(() => {
            markers = [];
            updateLineYs();
        }, clearMarkersDelay * 1000);
    }
    /**
     * @param {KeyboardEvent} event
     */
    function keydownHandler(event) {
        if (event.key == "m" || event.key == " ") {
            mark();
        } else if (event.key == "c" || event.key == "Escape" || event.key == "r") {
            clearMarkers();
        }
    }
    onMount(() => {
        document.addEventListener("keydown", keydownHandler);
        return () => document.removeEventListener("keydown", keydownHandler);
    })

    // getting measurements
    /** @type {HTMLDivElement} */
    let timerlineMarkerComponent;
    let width = 0;
    let height = 0;
    /** @type {number[]} */
    let blinkerXs = [];
    let blinkerWidth = 0;
    function updateMeasurements() {
        width = timerlineMarkerComponent.clientWidth;
        height = timerlineMarkerComponent.clientHeight;
        blinkerXs = Array.from(document.getElementsByClassName("frameBlinker")).map(b =>
            b.getBoundingClientRect().x + b.getBoundingClientRect().width / 2
        );
        blinkerWidth = document.getElementsByClassName("frameBlinker")[0]?.getBoundingClientRect().width || 0;
        updateLineYs();
        updateSVGs();
    }
    onMount(() => {
        updateMeasurements();
        window.addEventListener("resize", updateMeasurements);
        return () => window.removeEventListener("resize", updateMeasurements);
    })

    // vertical spacing
    const verticalSpacing = 20; // vh
    const lineSpacing = 10;  // vh
    let strokeWidth = 5; // px
    /** @type {number[]} */
    let lineYs = [];
    /**
     * @param {number} lineNumber
     * @returns {number}
     */
    function getLineY(lineNumber) {
        let lsp = lineSpacing * height / 100;
        const vsp = verticalSpacing * height / 100;
        if (markers.length * (strokeWidth + lsp) + 2 * vsp > height) {
            lsp = (height - 2 * vsp - markers.length * strokeWidth) / (markers.length - 1);
        }
        return vsp + strokeWidth / 2 + lineNumber * (lsp + strokeWidth);
    }
    function updateLineYs() {
        lineYs = Array.from({ length: markers.length }, (_, i) => getLineY(i));
    }

    let markerWidth = 29; // px
    let markerStrokeWidth = 6; // px

    function updateSVGs() {
        markerWidth = 0.02 * width;
        markerStrokeWidth = 0.005 * width;
        strokeWidth = 0.003 * width;
    }
</script>

<div id="timelineMarker" bind:this={timerlineMarkerComponent}>
    <svg class="timeline" viewBox={`0 0 ${width} ${height}`} xmlns="http://www.w3.org/2000/svg">
        {#if blinkerXs.length > 0}
            {#each markers as mks, i}
                <line class="lines"
                    x1={blinkerXs[0] - blinkerWidth*0.2} y1="0" x2={blinkerXs[blinkerXs.length-1] + blinkerWidth*0.2} y2="0"
                    stroke-width={strokeWidth}
                    style={`transform: translateY(${lineYs[i]}px); stroke-dasharray: ${width}px; stroke-dashoffset: ${width}px;`}
                />
                {#each mks as mk}
                    <g class="markerWrapper"
                        style={`transform: translate(${blinkerXs[mk] - markerWidth / 2}px, ${lineYs[i] - markerWidth / 2}px)`}
                    >
                        <rect class="markers"
                            width={markerWidth} height={markerWidth}
                            rx={markerWidth * 0.3} ry={markerWidth * 0.3}
                            stroke-width={markerStrokeWidth}
                        />
                    </g>
                {/each}
            {/each}
        {/if}
    </svg>
</div>

<style>
    #timelineMarker {
        --stroke-color: #CDCDD4;
        --marker-color: #ABC9D7;
    }

    #timelineMarker {
        width: 100%;
        flex-grow: 1;
        overflow: hidden;
    }

    .lines {
        stroke: var(--stroke-color);
        stroke-linecap: round;
        transition: transform 0.5s;
        animation-duration: 1s;
        animation-name: lineAnimation;
        animation-iteration-count: 1;
        animation-timing-function: "cubic-bezier(.08,.76,.33,.94)";
        animation-fill-mode: forwards;
    }
    @keyframes lineAnimation {
        to {
            stroke-dashoffset: 0;
        }
    }

    .markerWrapper {
        transition: transform 0.5s;
    }

    .markers {
        fill: var(--marker-color);
        stroke: var(--background-color);
        stroke-linejoin: bevel;
        transform-box: fill-box;
        transform-origin: center;
        animation-duration: 0.5s;
        animation-name: markerAnimation;
        animation-iteration-count: 1;
        animation-timing-function: "cubic-bezier(.08,.76,.33,.94)";
        animation-fill-mode: forwards;
    }
    @keyframes markerAnimation {
        from {
            transform: scale(0) rotate(-45deg);
        }
        to {
            transform: scale(1) rotate(45deg);
        }
    }
</style>