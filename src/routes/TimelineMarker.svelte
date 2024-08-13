<script>
  import { onMount } from "svelte";
  import BlinkerRow from "./BlinkerRow.svelte";

  /** @type {BlinkerRow} */
  export let blinkerRowComponent;

  // marking
  /** @type {number[][]} */
  let markers = [];
  let startCycle = 0;
  const lineLimit = 10;
  let clearAnimLock = false;
  /** @type {number[][]} */
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
      if (tempMarkers[lineNumber].includes(blinkerRowComponent.getCurrent())) return;
      tempMarkers[lineNumber].push(blinkerRowComponent.getCurrent());
      if (!markers[lineNumber].includes(blinkerRowComponent.getCurrent())) {
        markers[lineNumber].push(blinkerRowComponent.getCurrent());
      } else {
        const marker = timerlineMarkerComponent.getElementsByClassName(
          `markers l_${lineNumber} m_${blinkerRowComponent.getCurrent()}`
        )[0];
        marker?.getAnimations().forEach((a) => a.cancel());
        marker?.animate(
          [{ transform: "scale(0) rotate(-45deg)" }, { transform: "scale(1) rotate(45deg)" }],
          {
            duration: 500,
            easing: "cubic-bezier(.08,.76,.33,.94)",
            fill: "forwards"
          }
        );
      }
    } else {
      if (lineNumber >= markers.length) {
        for (let i = markers.length; i < lineNumber + 1; i++) {
          markers.push([]);
        }
      }
      if (markers[lineNumber].includes(blinkerRowComponent.getCurrent())) return;
      markers[lineNumber].push(blinkerRowComponent.getCurrent());
    }
    markers = markers;
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
    Array.from(timerlineMarkerComponent.getElementsByClassName("markers")).forEach((m, i) => {
      m.animate(
        [{ transform: "scale(1) rotate(45deg)" }, { transform: "scale(0) rotate(-45deg)" }],
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
    updateLineYs();
    updateSVGs();
  }
  onMount(() => {
    window.addEventListener("resize", updateMeasurements);
    window.addEventListener("touchstart", mark);
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
        {#each mks as mk (mk)}
          <g
            class="markerWrapper"
            style={`transform: translate(${blinkerXs[mk] - markerWidth / 2}px, ${lineYs[i] - markerWidth / 2}px)`}
          >
            <rect
              class={`markers l_${i} m_${mk}`}
              width={markerWidth}
              height={markerWidth}
              rx={markerWidth * 0.3}
              ry={markerWidth * 0.3}
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
