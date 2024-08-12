<script>
	export let fps = 12;
	export let interval = 3;
</script>

<h1 id="title">{fps} FPS, INTV {interval}</h1>

<style>
	#title {
		--font-color: #babac1;
	}

	#title {
		font-size: 3rem;
		font-family: 'Roboto', sans-serif;
		font-weight: 700;
		color: var(--font-color);
	}

	@media screen and (max-width: 700px) {
		#title {
			font-size: 2rem;
		}
	}
</style>
