<script>
  import world from "$data/world-110m.json";
  import * as topojson from "topojson-client";
  import { geoOrthographic, geoPath, geoCentroid } from "d3-geo";
  import { scaleLinear } from "d3-scale";
  import { max, sort } from "d3-array";
  import { timer } from "d3-timer";

  import Glow from "$components/Glow.svelte";
  import Tooltip from "$components/Tooltip.svelte";
  import Legend from "$components/Legend.svelte";

  import data from "$data/data.json";
  import { onMount } from "svelte";
  import { drag } from "d3-drag";
  import { select } from "d3-selection";
  import { spring } from "svelte/motion";
  import { draw } from "svelte/transition";
    

  // console.log({ world });

  let countries = topojson.feature(world, world.objects.countries).features;

  let borders = topojson.mesh(
    world,
    world.objects.countries,
    (a, b) => a !== b,
  );

  //Restructure countries array to include pop data
  countries.forEach((country) => {
    const metadata = data?.find((d) => d.id === country.id);
    if (metadata) {
      country.population = metadata.population;
      country.country = metadata.country;
    }
  });

  let width = 400;
  $: height = width;

  $: projection = geoOrthographic()
    .scale(width / 2)
    .rotate([$xRotation, $yRotation, 0])
    .translate([width / 2, height / 2]);

  $: path = geoPath(projection);

  const colorScale = scaleLinear()
    .domain([0, max(data, (d) => d.population)])
    .range(["#26362e", "#0DCC6C"]);

  let xRotation = spring(0, { stiffness: 0.08, damping: 0.4 });
  let yRotation = spring(0, { stiffness: 0.17, damping: 0.7 });
  let degreesPerFrame = 0.5;

  const t = timer((elapsed) => {
    if (dragging || tooltipData) return;
    $xRotation += degreesPerFrame;
    // if (elapse > 200) t.stop();
  }, 0);

  let globe;
  let dragging = false;

  onMount(() => {
    const element = select(globe);
    element.call(
      drag()
        .on("drag", (event) => {
          console.log("I am being dragged");
          $xRotation = $xRotation + event.dx * 0.5;
          $yRotation = $yRotation - event.dy * 0.5;
          dragging = true;
        })
        .on("end", () => {
          dragging = false;
        }),
    );
  });

  let tooltipData;

  $: if (tooltipData) {
    const center = geoCentroid(tooltipData);
    $xRotation = -center[0];
    $yRotation = -center[1];
    // projection.rotate([-center[0], -center[1]])
  }
</script>

<div class="chart-container" bind:clientWidth={width}>
  <h1>The World at a Glance</h1>
  <h2>Population by Country, 2021</h2>
  <svg {width} {height} bind:this={globe} class:dragging>

    <Glow />

    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <circle
      r={width / 2}
      cx={width / 2}
      cy={height / 2}
      fill="#1c1c1c"
      filter="url(#glow)"
      on:click={() => {
        tooltipData = null;
      }}
      on:focus={() => {
        tooltipData = null;
      }}
    />

    {#each countries.sort((a, b) => b.population - a.population) as country}
    <!-- console.log({country}); -->
      <!-- svelte-ignore a11y-click-events-have-key-events -->
      <!-- svelte-ignore a11y-no-noninteractive-tabindex -->
      <path
        d={path(country)}
        fill={colorScale(country?.population || 0)}
        stroke="none"
        on:click={() => {
          tooltipData = country;
        }}
        on:focus={() => {
          tooltipData = country;
        }}
        tabindex="0"
      />
    {/each}

    <path d={path(borders)} fill="none" stroke="black" />

    {#if tooltipData}
      {#key tooltipData.id}
        <path
          d={path(tooltipData)}
          fill="transparent"
          stroke="white"
          stroke-width="2"
          in:draw
        />
      {/key}
    {/if}
  </svg>
  <Tooltip data={tooltipData} />
  <Legend {colorScale} data={tooltipData}/>
</div>

<style>
  :global(body) {
    background: rgb(40, 40, 40);
  }

  .chart-container {
    max-width: 468px;
    margin: 0 auto;
  }

  svg {
    overflow: visible;
  }

  .dragging {
    cursor: grabbing;
    
  }

  path {
    cursor: pointer;
  }

  path:focus{
    outline: none;
  }

  path:focus{
    outline: none;
  }

  circle:focus{
    outline: none;
  }

  h1, h2{
    color: white;
    text-align: center;
  }

  h1{
    font-size: 1.75rem;
    font-weight: 800;
    margin-bottom: 0.4rem;
    margin-top: 0.4rem;
  }

  h2{
    font-size: 1.25rem;
    font-weight: 200;
    margin-bottom: 1.5rem;
  }

</style>
