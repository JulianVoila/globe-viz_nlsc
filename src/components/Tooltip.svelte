<script>
    import { format } from "d3-format";
    import { fade } from "svelte/transition";

    export let data;

    const suffixFormat = (d) => format(".2s")(d).replace("G", "B");
</script>

{#if data?.country}
    <!-- {#key data.id} -->
        <div in:fade>
            <h2>{data.country}</h2>
            <h3>Population of {suffixFormat(data.population)}</h3>
        </div>
    <!-- {/key} -->
{/if}

<style>
    div {
        position: absolute;
        bottom: 30px;
        right: 0;
        text-align: right;
    }

    h2 {
        font-size: 1.5rem;
        font-weight: 700;
        color: white;
    }

    h3 {
        font-size: 1.15rem;
        font-weight: 200;
        color: rgba(255, 255, 255, 0.7);
    }
</style>
