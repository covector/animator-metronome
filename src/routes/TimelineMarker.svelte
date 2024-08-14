<script>
  import { onMount } from "svelte";
  import BlinkerRow from "./BlinkerRow.svelte";

  export let fps = 12;
  export let lineLimit = 10;
  /** @type {BlinkerRow} */
  export let blinkerRowComponent;
  export let showAllDelta = false;
  export let hideDelta = false;
  export let noGrid = false;

  // marking
  /** @type {{frame: number, delta: number}[][]} */
  let markers = [];
  /**
   * @param {{frame: number, delta: number}[][]} mks
   * @param {number} line
   * @param {number} frame
   * @returns {number}
   */
  function getLastDelta(mks, line, frame) {
    for (let i = line; i >= 0; i--) {
      if (mks[i].length > 0) return line * fps + frame - i * fps - mks[i][mks[i].length - 1].frame;
    }
    return -1;
  }
  let startCycle = 0;
  let clearAnimLock = false;
  /** @type {{frame: number, delta: number}[][]} */
  let tempMarkers = [];
  export function mark() {
    if (animLock) return;
    updateMeasurements();
    if (markers.length === 0) {
      startCycle = 0;
      blinkerRowComponent.resetCycle();
    }
    let lineNumber = blinkerRowComponent.getCurrentCycle() - startCycle;
    if (markers.length != 0 && lineNumber >= lineLimit) {
      startCycle = 0;
      blinkerRowComponent.resetCycle();
      lineNumber = 0;
      clearAnimLock = true;
      animateOut(true);
    }
    if (clearAnimLock) {
      if (lineNumber >= tempMarkers.length) {
        for (let i = tempMarkers.length; i < lineNumber + 1; i++) {
          tempMarkers.push([]);
          const line = timerlineMarkerComponent.getElementsByClassName("lines l_" + i)[0];
          line?.getAnimations().forEach((a) => a.cancel());
          line?.animate([{ strokeDashoffset: width }, { strokeDashoffset: 0 }], {
            duration: 1000,
            easing: "cubic-bezier(.08,.76,.33,.94)",
            fill: "forwards"
          });
        }
      }
      if (
        tempMarkers[lineNumber].findIndex((m) => m.frame == blinkerRowComponent.getCurrent()) != -1
      )
        return;
      tempMarkers[lineNumber].push({
        frame: blinkerRowComponent.getCurrent(),
        delta: getLastDelta(tempMarkers, lineNumber, blinkerRowComponent.getCurrent())
      });
      // if (markers[lineNumber].findIndex(m => m.frame == blinkerRowComponent.getCurrent()) == -1) {
      //   markers[lineNumber].push({...tempMarkers[lineNumber][tempMarkers[lineNumber].length - 1]});
      // } else {
      //   const marker = timerlineMarkerComponent.getElementsByClassName(
      //     `markers l_${lineNumber} m_${blinkerRowComponent.getCurrent()}`
      //   )[0];
      //   marker?.getAnimations().forEach((a) => a.finish());
      //   marker?.animate(
      //     [{ transform: "scale(0) rotate(-45deg)" }, { transform: "scale(1) rotate(45deg)" }],
      //     {
      //       duration: 500,
      //       easing: "cubic-bezier(.08,.76,.33,.94)",
      //       fill: "forwards"
      //     }
      //   );
      // }
      // markers = markers;
    } else {
      if (lineNumber >= markers.length) {
        for (let i = markers.length; i < lineNumber + 1; i++) {
          markers.push([]);
        }
      }
      if (markers[lineNumber].findIndex((m) => m.frame == blinkerRowComponent.getCurrent()) != -1)
        return;
      markers[lineNumber].push({
        frame: blinkerRowComponent.getCurrent(),
        delta: getLastDelta(markers, lineNumber, blinkerRowComponent.getCurrent())
      });
      markers = markers;
    }
    updateLineYs();
  }
  const clearMarkersDelay = 0.5; // seconds
  let animLock = true;
  export function clearMarkers() {
    if (animLock) return;
    if (markers.length === 0) return;
    animLock = true;
    animateOut();
  }
  function animateOut(hack = false) {
    // animate lines
    Array.from(timerlineMarkerComponent.getElementsByClassName("lines")).forEach((l, i) => {
      l.animate([{ strokeDashoffset: 0 }, { strokeDashoffset: width }], {
        duration: clearMarkersDelay * 1000,
        easing: "cubic-bezier(.59,.34,.33,.94)",
        fill: "forwards"
      });
    });
    // animate markers
    Array.from(timerlineMarkerComponent.getElementsByClassName("markerWrapper")).forEach((m, i) => {
      m.getElementsByClassName("markers")[0]?.animate(
        [{ transform: "scale(1) rotate(45deg)" }, { transform: "scale(0) rotate(-45deg)" }],
        {
          duration: clearMarkersDelay * 1000,
          easing: "cubic-bezier(.59,.34,.33,.94)",
          fill: "forwards"
        }
      );
      m.getElementsByClassName("delta")[0]?.animate(
        [
          { opacity: 1, transform: "translate(0, 0)" },
          { opacity: 0, transform: "translate(0, 5px)" }
        ],
        {
          duration: clearMarkersDelay * 1000,
          easing: "cubic-bezier(.59,.34,.33,.94)",
          fill: "forwards"
        }
      );
    });
    setTimeout(() => {
      if (hack) {
        markers = tempMarkers;
        tempMarkers = [];
        clearAnimLock = false;
      } else {
        markers = [];
        animLock = false;
      }
      updateLineYs();
    }, clearMarkersDelay * 1000);
  }
  /**
   * @param {KeyboardEvent} event
   */
  function keydownHandler(event) {
    if (event.repeat) return;
    if (event.key == "m" || event.key == " " || event.key == "Enter") {
      mark();
    } else if (event.key == "c" || event.key == "Escape" || event.key == "r") {
      clearMarkers();
    }
  }
  onMount(() => {
    document.addEventListener("keydown", keydownHandler);
    return () => document.removeEventListener("keydown", keydownHandler);
  });

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
    blinkerXs = Array.from(document.getElementsByClassName("frameBlinker")).map(
      (b) => b.getBoundingClientRect().x + b.getBoundingClientRect().width / 2
    );
    blinkerWidth =
      document.getElementsByClassName("frameBlinker")[0]?.getBoundingClientRect().width || 0;
    updateSVGs();
    updateLineYs();
  }
  onMount(() => {
    window.addEventListener("resize", updateMeasurements);
    window.addEventListener("touchstart", mark, { passive: true });
    updateMeasurements();
    setTimeout(() => {
      animLock = false;
    }, 1000);
    return () => {
      window.removeEventListener("resize", updateMeasurements);
      window.removeEventListener("touchstart", mark);
    };
  });

  // vertical spacing
  let verticalSpacing = 20; // vh
  let lineSpacing = 10; // vh
  let strokeWidth = 5; // px
  /** @type {number[]} */
  let lineYs = [];
  /**
   * @param {number} lineNumber
   * @returns {number}
   */
  function getLineY(lineNumber) {
    let lsp = (lineSpacing * height) / 100;
    const vsp = (verticalSpacing * height) / 100;
    if (markers.length * (strokeWidth + lsp) - lsp + 2 * vsp > height) {
      lsp = (height - 2 * vsp - markers.length * strokeWidth) / (markers.length - 1);
    }
    return vsp + strokeWidth / 2 + lineNumber * (lsp + strokeWidth);
  }
  function updateLineYs() {
    if (width > 700) {
      verticalSpacing = 20;
      lineSpacing = 10;
    } else {
      verticalSpacing = 15;
      lineSpacing = 8;
    }
    lineYs = Array.from({ length: markers.length }, (_, i) => getLineY(i));
  }

  let markerWidth = 29; // px
  let markerStrokeWidth = 6; // px

  function updateSVGs() {
    if (width > 1000) {
      markerWidth = 0.02 * width;
      markerStrokeWidth = 0.005 * width;
      strokeWidth = 0.003 * width;
    } else {
      markerWidth = 20;
      markerStrokeWidth = 4;
      strokeWidth = 3;
    }
  }
</script>

<div id="timelineMarker" bind:this={timerlineMarkerComponent}>
  <svg class="timeline" viewBox={`0 0 ${width} ${height}`} xmlns="http://www.w3.org/2000/svg">
    {#if blinkerXs.length > 0}
      {#each markers as mks, i (i)}
        <line
          class={`lines l_${i}`}
          x1={blinkerXs[0] - blinkerWidth * 0.2}
          y1="0"
          x2={blinkerXs[blinkerXs.length - 1] + blinkerWidth * 0.2}
          y2="0"
          stroke-width={strokeWidth}
          style={`transform: translateY(${lineYs[i]}px); stroke-dasharray: ${width}px; stroke-dashoffset: ${width}px;`}
        />
      {/each}
      {#if !noGrid}
        {#each blinkerXs as b, i (i)}
          <line class="lineUnit" x1={b} y1="0" x2={b} y2={height} stroke-width={strokeWidth} />
        {/each}
      {/if}
      {#each markers as mks, i (i)}
        {#each mks as mk, j (mk)}
          <g
            class="markerWrapper"
            style={`transform: translate(${blinkerXs[mk.frame] - markerWidth / 2}px, ${lineYs[i] - markerWidth / 2}px)`}
          >
            <rect
              class={`markers l_${i} m_${mk.frame}`}
              width={markerWidth}
              height={markerWidth}
              rx={markerWidth * 0.3}
              ry={markerWidth * 0.3}
              stroke-width={markerStrokeWidth}
            />
            {#if !hideDelta && ((showAllDelta && mk.delta != -1) || mk.delta > fps / 2 || (mk.delta != -1 && j == 0))}
              <text
                class="delta"
                x={mk.frame == 0
                  ? blinkerXs[blinkerXs.length - 1] -
                    blinkerXs[mk.frame] +
                    blinkerWidth * 0.2 +
                    markerWidth * 0.6
                  : -markerWidth * 0.08}
                y={markerWidth * 0.15 + (mk.frame == 0 ? lineYs[i - 1] - lineYs[i] : 0)}
                dominant-baseline="middle"
                text-anchor="end"
                font-size={markerWidth * 0.5}
              >
                {mk.delta}
              </text>
            {/if}
          </g>
        {/each}
      {/each}
    {/if}
  </svg>
</div>

<style>
  #timelineMarker {
    --stroke-color: #cdcdd4;
    --marker-color: #abc9d7;
  }

  #timelineMarker {
    width: 100%;
    max-height: 70vh;
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

  .lineUnit {
    stroke: var(--background-color);
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

  .markerWrapper text {
    font-family: "Roboto", sans-serif;
    fill: var(--marker-color);
    font-weight: 700;
    animation: textAnimation 0.5s cubic-bezier(0.08, 0.76, 0.33, 0.94) forwards;
    user-select: none;
  }
  @keyframes textAnimation {
    from {
      opacity: 0;
      transform: translate(0, 20px);
    }
    to {
      opacity: 1;
      transform: translate(0, 0);
    }
  }
</style>
