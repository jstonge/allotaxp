<script>
// @ts-nocheck

    import { scaleBand, rollup, range, InternSet, scaleLinear, extent, schemePiYG, map } from "d3"
    export let test_elem_1;
    export let test_elem_2;

   
    const setdiff = (x,y) => {
        let a = new Set(x);
        let b = new Set(y);
        return new Set(
        [...a].filter(x => !b.has(x)));
    } 

    const union = (x,y) => {
        let a = new Set(x);
        let b = new Set(y);
        return new Set([...a, ...b]);
    }

    $: types_1 = test_elem_1.map(d => d.types)
    $: types_2 = test_elem_2.map(d => d.types)
    
    $: union_types = union(types_1, types_2)
    $: tot_types = types_1.length+types_2.length


    export let DiamondWidth;
    export let DiamondHeight;

    let width = 200;
    let margin = {  right: 40, top: 30, left: 40, bottom: 10 };
    let yPadding = 0.5;

    let colors = ["lightgrey", "lightblue"]

    $: dat = [ 
            { y_coord: "total count",     frequency: +(types_2.length / tot_types).toFixed(3) },
            { y_coord: "total count",     frequency: -(types_1.length / tot_types).toFixed(3) },
            { y_coord: "all names",       frequency: +(types_2.length / union_types.size).toFixed(3) },
            { y_coord: "all names",       frequency: -(types_1.length / union_types.size).toFixed(3) },
            { y_coord: "exclusive names", frequency: +(setdiff(types_2, types_1).size / types_2.length).toFixed(3) },
            { y_coord: "exclusive names", frequency: -(setdiff(types_1, types_2).size / types_1.length).toFixed(3) } 
        ]
    
    $: X = dat.map(d => d.frequency)
    $: Y = dat.map(d => d.y_coord);
    $: xRange = [margin.left, width - margin.right]
    $: xDomain = extent(X);
    $: yDomain = new InternSet(Y);

    $: height = Math.ceil((yDomain.size + yPadding) * 25) + margin.top + margin.bottom;

    $: I = range(X.length).filter(i => yDomain.has(Y[i]));
    $: YX = rollup(I, ([i]) => X[i], i => Y[i]);


    $: xScale = scaleLinear(xDomain, xRange)
    $: yScale = scaleBand()
                        .domain(yDomain)
                        .range([margin.top, height - margin.bottom]).padding(yPadding);
  
    $: format = xScale.tickFormat(100, "%");
    
</script>

<g class="balance-chart" transform="translate({DiamondWidth-75}, {DiamondHeight+75})">
    {#each dat as d, i}
    <rect
        x={Math.min(xScale(0), xScale(d.frequency))}
        y={yScale(d.y_coord)}
        fill={colors[X[i] > 0 ? colors.length - 1 : 0]}
        width={ Math.abs(xScale(d.frequency) - xScale(0))}
        height={ yScale.bandwidth() }
    />
    <text
        x={xScale(X[i]) + Math.sign(X[i] - 0) * 4}
        y={yScale(Y[i]) + yScale.bandwidth() / 2}
        opacity="0.5"
        dy="0.35em"
        font-size=10
        text-anchor = {d.frequency < 0 ? "end" : "start"}
    >{format(Math.abs(d.frequency))}</text>
    {/each}
    {#each yScale.domain() as text,i}
        <g class="diverging-ticks">
            <text
                x={xScale(0)}
                y={yScale(text) + yScale.bandwidth() / 2}
                dy="-.9em"
                dx="-3em"
                font-size="10"
            >{text}</text>
        </g>
    {/each}
</g>

    