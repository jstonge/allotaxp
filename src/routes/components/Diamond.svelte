<script>
    // @ts-nocheck

    import * as d3 from "d3";

	import AxisY from './AxisY.svelte';
	import AxisX from './AxisX.svelte';
    import Grid from './Grid.svelte'
    
    import { rin } from "./utils_helpers.js"

    export let diamond_dat;
    export let relevant_types;
    export let max_rank_raw;
    export let height;
    export let title;
    
    const margin = { inner: 160, diamond: 40 };
    
    $: innerHeight = height - margin.inner;   
    $: diamondHeight = innerHeight - margin.diamond;

    $: ncells = d3.max(max_rank_raw)
	$: max_rank = d3.max(diamond_dat, (d) => d.rank_L[1]);
    $: rounded_max_rank = 10**Math.ceil(Math.max(Math.log10(max_rank)))
    $: xyDomain = [1, rounded_max_rank];
    // $: bin_size = 1.5;
    
    // $: buckets = d3.range(0, ncells, bin_size)

    $: xy  = d3.scaleBand().domain(max_rank_raw).range([0, diamondHeight])
    $: wxy  = d3.scaleBand().domain(max_rank_raw).range([0, innerHeight])
	
    $: logScale = d3.scaleLog().domain(xyDomain).range([0, innerHeight]).nice()
    $: linScale = d3.scaleLinear().domain([0,ncells-1]).range([0, innerHeight])
    $: color_scale = d3.scaleSequentialLog().domain([rounded_max_rank, 1]).interpolator(d3.interpolateInferno)     

    $: blue_triangle = [[innerHeight, innerHeight], [0, 0], [0, innerHeight]].join(" ")
    $: grey_triangle = [[innerHeight, innerHeight], [0, 0], [innerHeight, 0]].join(" ")
    
    function filter_labs(d, relevant_types) {
        return rin(relevant_types, d.types.split(",")).some((x) => x === true)
    }
</script>


        <g class='inner-chart' transform="translate(360, 0) scale (-1,1) rotate(45) translate({margin.inner/2}, {margin.inner/2})">
            <polygon points={blue_triangle} fill="#89CFF0" fill-opacity=0.2 stroke="black" stroke-width=0.5/>
            <polygon points={grey_triangle} fill="grey" fill-opacity=0.2 stroke="black" stroke-width=0.5/>
            
            <Grid  height={innerHeight} {wxy} {ncells} scale={linScale}></Grid>
            <AxisY height={innerHeight} scale={logScale} {title}/>
            <AxisX height={innerHeight} scale={logScale} {title}/>
            
            {#each diamond_dat as d}
            <rect
                x={xy(d.x1)}
                y={xy(d.y1)}
                width={xy.bandwidth()}
                height={xy.bandwidth()}
                fill={color_scale(d.value)}
                opacity={d.value === null ? 0 : 1.}
                stroke="black"
                stroke-width=0.2
            />
            {/each}
            {#each diamond_dat.filter(d => filter_labs(d, relevant_types)) as d}
                <g class="diamond-lab" 
                   transform="
                    scale(1,-1) 
                    rotate(-90) 
                    rotate(-45, {xy(d.x1)}, {xy(d.y1)}) 
                    translate({d.which_sys === "right" ? xy(Math.sqrt(d.cos_dist))*1.5 : -xy(Math.sqrt(d.cos_dist))*1.5}, 0)
                   ">
                   <text
                        x={xy(d.x1)}
                        y={Number.isInteger(d.coord_on_diag) ? xy(d.y1) : xy(d.y1)-1}
                        dy={20}
                        font-size="10"
                        text-anchor={d.x1 - d.y1 <= 0 ? "start" : "end"}
                   >{d.types.split(",")[0]}</text>
                </g>
                {/each}
    </g>


