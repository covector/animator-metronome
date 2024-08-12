<script>
	export let fps = 12;
	export let interval = 3;
</script>

<h1 id="title">{fps} FPS, INTV {interval}</h1>

<style>
	#title {
		--font-color: #c1c1c9;
	}

	#title {
		font-size: 3rem;
		font-family: 'Roboto', sans-serif;
		font-weight: 700;
		color: var(--font-color);
		opacity: 0;
		transform: translateY(3rem);
		animation: fadeIn 1s 0.7s forwards;
		user-select: none;
	}

	@keyframes fadeIn {
		from {
			opacity: 0;
			transform: translateY(3rem);
		}
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}

	@media screen and (max-width: 700px), 
		screen and (max-height: 700px) {
		#title {
			font-size: 2rem;
		}
	}
</style>
