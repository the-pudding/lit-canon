<script>
    import Graphic from "$components/Graphic.svelte"
    import { onMount } from "svelte";

    let viewportWidth = 0;

    console.log("hiii")
    let mounted = false;

    onMount(async () => {
        console.log(viewportWidth, "viewwed")
        mounted = true;
        
    })


</script>
<!-- <Demo /> -->


<div class="thing" bind:clientWidth={viewportWidth}>
    {#if mounted}
        <Graphic viewportWidth={viewportWidth}/>
    {/if}
</div>


<style>
	.thing {
		width: 100%;
		height: 100%;
	}
</style>