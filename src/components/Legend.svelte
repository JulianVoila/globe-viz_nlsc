<script>
    import { format } from "d3-format";
    import { fade } from "svelte/transition";

    export let colorScale;
    export let data;

    const minColor = colorScale.range()[0];
    const maxColor = colorScale.range()[1];

    const minValue = colorScale.domain()[0];
    const maxValue = colorScale.domain()[1];

    const suffixFormat = (d) => format(".2s")(d).replace("G", "B");

    $: percentOfMax = (data?.population / maxValue) * 100;
</script>

<div class="legend">
    <span class="label">{minValue}</span>
    <div
        class="bar"
        style:background="linear-gradient( to right,
        {minColor} 0%,
        {maxColor} 100%)"
    >
        {#if percentOfMax}
            <span class="line" style:left={percentOfMax + "%"} 
            transition:fade/>
        {/if}
    </div>
    <span class="label">{suffixFormat(maxValue)}</span>
</div>

<style>
    .legend {
        display: flex;
        flex-direction: row;
        column-gap: 6px;
        margin-top: 10px;
        position: relative;
    }

    .bar {
        height: 15px;
        width: 100%;
        position: relative;
    }

    .label {
        color: white;
        font-size: 0.85rem;
        user-select: none;
    }

    .line {
        position: absolute;
        top: 0;
        height: 15px;
        width: 2px;
        background-color: white;
        transition: left 500ms cubic-bezier(1, 0, 0, 1);
    }
</style>
