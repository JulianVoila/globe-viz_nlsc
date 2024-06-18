<script>
    import world from "$data/world-110m.json";
    import * as topojson from "topojson-client";
    import { geoOrthographic, geoPath } from "d3-geo";
    
    import Glow from "$components/Glow.svelte";

  console.log({world})

  let countries = topojson.feature(world, 
    world.objects.countries).features

  let borders = topojson.mesh(world,
    world.objects.countries, (a,b) => a !== b);



    

    let width = 400;
    $: height = width;

    $: projection = geoOrthographic()
    .scale(width/2)
    .rotate([0, 0])
    .translate([width/2, height/2])

    $: path = geoPath(projection);

</script>

<!-- {#each countries as country }
<p> {country.geometry.coordinates} </p>
{/each} -->

<div class="chart-container" bind:clientWidth={width}>
<svg width={width} height={height}>
  <Glow/>
  <circle r={width / 2} cx={width / 2} cy={height / 2} fill="#1c1c1c" filter="url(#glow)"/>

{#each countries as country }
<path d={path(country)} fill="#26362E" stroke="none"/>
{/each}

  <path d={path(borders)} fill="none" stroke="black"/>
</svg>
</div>

<style>
  :global(body){
    background: rgb(40,40,40);
  }
  
  .chart-container {
    max-width: 468px;
    margin: 0 auto;
  }

  svg{
    overflow: visible;
  }

</style>