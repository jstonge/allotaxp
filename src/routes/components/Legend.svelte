<script>
// @ts-nocheck

  import { scaleOrdinal, range, interpolateInferno, scaleBand, max, rgb, descending, scaleLog } from "d3";
  export let diamond_dat; // to know the scale of the legend
  export let DiamondHeight;

  const N_CATEGO = 20
  const myramp = range(N_CATEGO).map(i => rgb(interpolateInferno(i / (N_CATEGO - 1))).hex())
  const color = scaleOrdinal(range(N_CATEGO), myramp)

  let height = 370;

  const margin = {  right: 40, top: 65, left: 10 };
  $: innerHeight = height - margin.top - margin.right;   
  $: max_rank = max(diamond_dat, (d) => d.rank_L[1]);
  
  // $: console.log(color.domain())

  $: y = scaleBand()
          .domain(color.domain().reverse())
          .rangeRound([0, innerHeight]);

  $: logY = scaleLog()
     .domain([1, 10**Math.ceil(Math.max(Math.log10(max_rank))-1)])
     .rangeRound([0, innerHeight]).nice();
  
   $: logFormat10 = logY.tickFormat()
   $: yTicks = logY.ticks()
</script>


        <g class="legend-container" transform="translate({margin.left}, {DiamondHeight-margin.top})">
          {#each color.domain() as d}
          <rect
              x={ 0 }
              y={ y(d) }
              width={ 14 }
              height={ 13 }
              fill={ color(d) }
              stroke="whitesmoke"
              stroke-width="1"
          ></rect>
          {/each}
        </g>
        {#each yTicks as tick, i}
          <g class="legend-text" transform="translate({margin.left}, {DiamondHeight+logY(tick)-margin.top})">
            <text
              font-size="10" 
              dy={yTicks.length-1 == i ? "-3" :  "13"}
              dx="20"
            >{ logFormat10(tick) }</text>
          </g>
        {/each}
