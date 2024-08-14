<script>
  import { onMount } from "svelte";
  import TimelineMarker from "./TimelineMarker.svelte";
  /** @type {TimelineMarker} */
  export let timelineMarkerComponent;

  const buttonWidth = 60;
  const iconWidth = 25;

  onMount(() => {
    if ("ontouchstart" in window || navigator?.maxTouchPoints > 0) {
      isTouch = true;
    }
  });
  let isTouch = false;
</script>

<div id="markerButtons" class={isTouch ? "isTouch" : ""}>
  <button
    aria-label="Add marker"
    on:mousedown={timelineMarkerComponent.mark}
    class="markButton"
    on:touchstart|stopPropagation
  >
    <svg
      width={buttonWidth}
      height={buttonWidth}
      viewBox={`0 0 ${buttonWidth} ${buttonWidth}`}
      xmlns="http://www.w3.org/2000/svg"
    >
      <rect
        x={(buttonWidth - iconWidth) / 2}
        y={(buttonWidth - iconWidth) / 2}
        width={iconWidth}
        height={iconWidth}
        rx={iconWidth * 0.2}
        ry={iconWidth * 0.2}
        stroke-width={iconWidth * 0.2}
      />
    </svg>
  </button>
  <button
    aria-label="Clear markers"
    on:mousedown={timelineMarkerComponent.clearMarkers}
    class="clearButton"
    on:touchstart|stopPropagation
  >
    <svg
      width={buttonWidth}
      height={buttonWidth}
      viewBox={`0 0 ${buttonWidth} ${buttonWidth}`}
      xmlns="http://www.w3.org/2000/svg"
    >
      <rect
        x={(buttonWidth - iconWidth) / 2}
        y={(buttonWidth - iconWidth) / 2}
        width={iconWidth}
        height={iconWidth}
        rx={iconWidth * 0.2}
        ry={iconWidth * 0.2}
        stroke-width={iconWidth * 0.2}
      />
      <rect
        class="cross"
        x={(buttonWidth - iconWidth * 0.35) / 2}
        y={buttonWidth * 0.07}
        width={iconWidth * 0.35}
        height={buttonWidth * 0.86}
        stroke-width={iconWidth * 0.15}
        transform="rotate(90 30 30)"
      />
    </svg>
  </button>
</div>

<style>
  #markerButtons {
    --button-background: white;
    --mark-button-color: #8eafe0;
    --clear-button-color: #dc5d4f;
    --mark-button-hover-color: #dde4f0;
    --clear-button-hover-color: #f4d5d1;
  }

  #markerButtons {
    position: fixed;
    display: flex;
    justify-content: center;
    bottom: 1.5rem;
    right: 1rem;
    gap: 1.5rem;
  }

  #markerButtons button {
    position: relative;
    background-color: var(--button-background);
    border: none;
    border-radius: 50%;
    padding: 0;
    line-height: 0;
    box-shadow: 0 0 0.5rem rgba(0, 0, 0, 0.06);
    overflow: hidden;
    -webkit-tap-highlight-color: transparent;
    transition:
      background-color 0.2s,
      filter 0.2s;
  }

  #markerButtons svg rect {
    fill: none;
    transform-origin: center;
    transform-box: fill-box;
    transform: rotate(45deg);
  }

  .markButton rect {
    stroke: var(--mark-button-color);
  }

  .clearButton rect {
    stroke: var(--clear-button-color);
  }

  #markerButtons .clearButton svg rect.cross {
    fill: var(--clear-button-color);
    stroke: var(--button-background);
    transition: stroke 0.2s;
  }

  #markerButtons button:hover {
    cursor: pointer;
  }

  #markerButtons:not(.isTouch) button:active {
    filter: brightness(0.9);
  }

  #markerButtons:not(.isTouch) button.markButton:hover {
    background-color: var(--mark-button-hover-color);
  }

  #markerButtons:not(.isTouch) button.clearButton:hover {
    background-color: var(--clear-button-hover-color);
  }
  #markerButtons:not(.isTouch) .clearButton:hover svg rect.cross {
    stroke: var(--clear-button-hover-color);
  }

  #markerButtons.isTouch button.markButton:active {
    background-color: var(--mark-button-hover-color);
  }

  #markerButtons.isTouch button.clearButton:active {
    background-color: var(--clear-button-hover-color);
  }
  #markerButtons.isTouch .clearButton:active svg rect.cross {
    stroke: var(--clear-button-hover-color);
  }
</style>
