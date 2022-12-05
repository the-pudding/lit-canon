<script>
	export let label;
	export let style = "inner";
	export let options = ["on", "off"];
	export let value = options[0];


	let checked = value// === options[0];

	const id = `toggle-${Math.floor(Math.random() * 1000000)}`;

	const handleClick = (event) => {
		const target = event.target;
		const state = target.getAttribute("aria-checked");
		console.log(checked)
		checked = state === "true" ? false : true;
		value = checked// ? options[0] : options[1];
	};
</script>

<div class="toggle toggle--{style}">
	<span class="label" {id}>{label}</span>
	<button
		role="switch"
		aria-checked={checked}
		aria-labelledby={id}
		on:click={handleClick}
	>
		{#if style === "inner"}
			{#each options as option}
				<span>{@html option}</span>
			{/each}
		{/if}
	</button>
</div>

<style>
	.toggle button,
	.label {
		font-family: inherit;
		font-size: 1em;
	}

	.label {
		font-family: "Inter";
		font-weight: 600;
		font-size: 12px;
		color: rgba(0,0,0,.3);
	}

	.toggle--inner [role="switch"][aria-checked="true"] :first-child,
	[role="switch"][aria-checked="false"] :last-child {
		/* display: inline-block;
		border-radius: 4px;
		background: var(--color-gray-900); */
		/* color: var(--color-gray-100); */
		border-bottom: 1px solid #B4B4B4;
		position: relative;
	}

	.toggle--inner [role="switch"][aria-checked="true"] :first-child:before, [role="switch"][aria-checked="false"] :last-child:before {
		content: '';
		position: absolute;
		top: -5px;
		left: 50%;
		width: 0; 
		height: 0; 
		border-left: 5px solid transparent;
		border-right: 5px solid transparent;
		border-top: 7px solid #AC1919;
		transform: translate(-50%,0);
	}

	.toggle--inner button {
		padding: 0.5em;
		font-size: 18px;
	}

	.toggle--inner button span {
		user-select: none;
		pointer-events: none;
		display: inline-block;
		line-height: 1;
		padding: 0.25em;
	}

	.toggle--inner button:focus {
		background: none;
		/* box-shadow: 0 0 4px 0 var(--color-focus); */
	}

	.toggle--slider {
		display: flex;
		align-items: center;
	}

	.toggle--slider button {
		width: 3.5em;
		height: 2em;
		position: relative;
		margin-left: 0.5em;
		background: var(--color-gray-300);
	}

	.toggle--slider button:focus {
		background: none;
		/* box-shadow: 0 0px 4px var(--color-focus); */
	}

	button {
		background: none;
	}

	.toggle--slider button::before {
		content: "";
		position: absolute;
		width: 1.5em;
		height: 1.5em;
		top: 0.25em;
		right: 1.75em;
	}

	.toggle--slider button[aria-checked="true"] {
		/* background-color: var(--color-gray-900); */
	}

	.toggle--slider button[aria-checked="true"]::before {
		transform: translateX(1.5em);
	}

	.toggle--slider button:focus {
		background: none;
		/* box-shadow: 0 0 4px 0 var(--color-focus); */
	}
</style>
