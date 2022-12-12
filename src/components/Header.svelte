<script>
	import wordmark from "$svg/wordmark.svg";
</script>

<header>
	<div class="wordmark">
		<a href="https://pudding.cool" target="_blank" aria-label="The Pudding">The Pudding</a>
	</div>
</header>

<style>
	.wordmark {
		margin: 0 auto;
		padding: 1em 0;
	}

	.wordmark a {
		border: none;
		display: block;
		color: var(--color-fg);
		text-transform: uppercase;
		letter-spacing: 3px;
		font-size: 18px;
		text-align: center;
	}

	.wordmark a:hover {
		background-color: transparent;
	}

	:global(.wordmark svg path) {
		fill: currentColor;
	}
</style>
