<script>
	import { onMount } from "svelte";
	import wordmark from "$svg/wordmark.svg";

	let localURL;
	let stories = [];

	const v = Date.now();
	const url = `https://pudding.cool/assets/data/stories.json?v=${v}`;

	const links = [
		{ name: "about", url: "https://pudding.cool/about" },
		{ name: "facebook", url: "https://facebook.com/pudding.viz/" },
		{ name: "twitter", url: "https://twitter.com/puddingviz/" },
		{
			name: "instagram",
			url: "https://www.instagram.com/the.pudding"
		},
		{ name: "patreon", url: "https://patreon.com/thepudding/" },
		{ name: "privacy", url: "https://pudding.cool/privacy/" },
		{ name: "newsletter", url: "https://thepuddingmail.substack.com" },
		{ name: "rss", url: "https://pudding.cool/feed/index.xml" }
	];

	onMount(async () => {
		localURL = window.location.href;
		const response = await fetch(url);
		const data = await response.json();
		stories = data.filter((d) => !localURL.includes(d.url)).slice(0, 4);
	});
</script>

<footer>
	<section class="stories">
		{#each stories as { hed, url, image }}
			<div class="story">
				<a href="https://pudding.cool/{url}">
					<img
						src="https://pudding.cool/common/assets/thumbnails/640/{image}.jpg"
						alt="thumbnail"
					/>
					<span>{hed}</span>
				</a>
			</div>
		{/each}
	</section>

</footer>

<style>
	footer {
		margin-top: 2rem;
	}

	a,
	a:visited,
	a:hover {
		color: #000;
		border: none;
		text-decoration: none;
	}

	.stories {
		margin: 0 auto;
		display: flex;
		flex-wrap: wrap;
		justify-content: space-between;
		max-width: 70em;
	}

	.story {
		display: block;
		width: calc(50% - 10px);
		border: none;
		margin-bottom: 3rem;
	}

	.story a {
		display: block;
		font-size: 18px;
		text-decoration: none;
	}

	.story span {
		display: block;
		margin-top: 1em;
		line-height: 1.2;
	}

	.wordmark {
		max-width: 10em;
		margin: 1em auto;
	}

	.about {
		margin: 3rem auto;
		margin-top: 0;
		text-align: center;
	}

	.links ul {
		display: flex;
		flex-wrap: wrap;
		justify-content: center;
	}

	.links li {
		display: flex;
		padding: 0.5em 1em;
	}

	.links a {
		display: flex;
		border: none;
		align-items: center;
		text-decoration: none;
	}

	.links span {
		margin-left: 0.5em;
	}

</style>
