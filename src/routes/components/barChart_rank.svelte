<script>
    import * as d3 from "d3";
    export let barData;
    export let height;
    
    import { fly } from 'svelte/transition';
    
	$: width = 640;   
    
    let margin = ({ top: 40, left: 40, right: 60, bottom: 70 })

    $: babynames = barData.map(d => d.types)

    $: xScale = d3.scaleBand()
                  .domain(babynames)
                  .range([margin.left, width - margin.right])
                  
    $: yScale = d3.scaleLinear()
                  .domain([0, d3.max(barData.map(d => d.counts))])
                  .range([height - margin.bottom, margin.top]);

    $: yTicks = yScale.ticks(8)
</script>

<div class="chart-container" bind:clientWidth={width}> 
    <svg {width} {height}>
        <g class="axis x" transform="translate(0, {yScale(0)})">
            {#each babynames as name, i}
                <g class="tick" transform="translate({xScale(name)}, 0)">
                    <text
                        dy=10
                        dx=10
                        text-anchor="start"
                        transform="rotate(45)"
                    >{name}</text>
                </g>
            {/each}
        </g>
        <g class="axis y">
            {#each yTicks as tick, i}
                <g class="tick" transform="translate(0, {yScale(tick)})">
                    <text
                        y={-3}
                        dx=10
                        dy=6
                        text-anchor="start"
                    >{tick/1000}K</text>
                    <line
                        x1={margin.left}
                        x2={width-margin.right}
                        y1={0}
                        y2={0}
                        stroke={i === 0 ? '#8f8f8f' : '#e5e7eb'}
                    ></line>
                </g>
            {/each}
        </g>
        {#each barData as d, i}
        <circle
            in:fly={{ y: 10, opacity: 0, duration: 500}}
            cx={xScale(d.types)}
            cy={yScale(d.counts)}
            r=4
            fill="midnightblue"
            opacity=0.8
            stroke="black"
        />
        {/each}
    </svg>
</div>


<style>
    .chart-container {
        position: relative;
    }

    circle {
        transition: r 300ms ease, opacity 500ms ease;
        cursor: pointer;
    }

    .tick {
        fill: hsla(212, 10%, 53%, 1);
        font-size: 12px
    }
</style>
