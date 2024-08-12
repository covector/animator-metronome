<script>
    /** @type {boolean} */
    export let primary = true;
    export let isBlinking = false;
</script>

<div class={`frameBlinker${isBlinking ? " blinking" : ""}${primary ? " primary" : ""}`}>
    <div ></div>
</div>

<style>
    .frameBlinker {
        --non-blink: white;
        --blink-primary: #75A3E7;
        --blink-secondary: #e7e7e7;
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
    }

    .frameBlinker.blinking div {
        background-color: var(--blink-secondary);
        transition: background-color 0.05s;
    }

    .frameBlinker.primary.blinking div {
        background-color: var(--blink-primary);
    }
</style>