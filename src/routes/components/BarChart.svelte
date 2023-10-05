<script>
    // @ts-nocheck

    import * as d3 from "d3";
    import diamond from "./diamond_count";

    export let barData;

    let margin = ({ top: 80, left: 140, right: 50, bottom: 10 })

    export let DashboardHeight;
    export let DashboardWidth;

	let width = 640;
    
    let yPadding = 0.2;
    
    // $: X = d3.map(barData, (d) => d.metric);
    $: Y = d3.map(barData, (d) => d.type);
    
    $: max_shift = d3.max(barData, (d) => Math.abs(d.metric))*1.5

    $: xDomain = [-max_shift, max_shift];
    $: yDomain = new d3.InternSet(Y);
    // $: I = d3.range(X.length).filter(i => yDomain.has(Y[i]));
    // $: YX = d3.rollup(I, ([i]) => X[i], i => Y[i]);
    
    // const height = 800;

    $: xRange = [DashboardWidth - width + margin.left, DashboardWidth - margin.right];
    $: yRange = [margin.top, DashboardHeight - margin.bottom];
    
    $: xScale = d3.scaleLinear().domain(xDomain).range(xRange)
    $: yScale = d3.scaleBand(yDomain, yRange).padding(yPadding);
        
    $: xTicks = xScale.ticks(width / 80)
    // $: yTicks = Array.from(yDomain)

    const colors = ["lightgrey", "lightblue"]

    // $: console.log(barData.map(d => d.metric > 0))
</script>


        <g class='barChart-container'>
            <g class='axis x' transform="translate(0, {margin.top})">
                {#each xTicks as tick}
                    <g class="tick">
                        <line 
                            x1={xScale(tick)} 
                            y1="0" 
                            x2={xScale(tick)}
                            y2={DashboardHeight - margin.top - margin.bottom} 
                            stroke="hsla(212, 10%, 53%, 1)"
                            stroke-opacity=0.2
                        >{tick}</line>
                        <text 
                        x={xScale(tick)} 
                        y="-12" 
                        font-size=0.8em
                        >{tick*100}%</text>
                    </g>
                {/each}
            </g>
            {#each barData as d, i}
                <rect
                    x={Math.min(xScale(0), xScale(d.metric))}
                    y={yScale(d.type)}
                    fill={colors[d.metric > 0 ? colors.length - 1 : 0]}
                    width={Math.abs(xScale(d.metric) - xScale(0))}
                    height={ yScale.bandwidth() }
                />
                <text 
                    x={Math.min(xScale(0), xScale(d.metric))}
                    y={yScale(d.type)}
                    dy="14"
                    font-size=0.7em
                >{d.type}</text>
            {/each}
        </g>


<style>
    .barChart-container {
        position: relative;
    }
</style>
