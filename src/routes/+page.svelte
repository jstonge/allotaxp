<script>
	// @ts-nocheck

	import Diamond from "./components/Diamond.svelte";
	import Wordshift from "./components/BarChart.svelte";
	import Legend from './components/Legend.svelte'
	import DivergingBarChart from './components/DivergingBarChart.svelte'
	import Tooltip from "./components/Tooltip.svelte";
	import MathJax from './MathJax.svelte';

	import { combElems, RTD, myDiamond, wordShift_dat } from "./components/combine_distributions.js"
	import { downloadChart, get_relevant_types } from "./components/utils_helpers"
	
	import { range, max, scaleBand } from 'd3'
	import { onMount } from "svelte";
	import Papa from 'papaparse';

	// diamond Stuff
	import * as d3 from "d3";
	import AxisY from './components/AxisY.svelte';
	import AxisX from './components/AxisX.svelte';
    import Grid from './components/Grid.svelte'
    
    import { rin } from "./components/utils_helpers.js"

	let files = [];
	let test_elem_1 = [];
	let test_elem_2 = [];
	let relevant_types = [];
	let hoveredData;
	let selected_sys1;
	let selected_sys2;

	const margin = { inner: 160, diamond: 40 };

	let DashboardHeight = 800;
	let DashboardWidth = 1200;
	let DiamondHeight = 600;
	let DiamondWidth = 600;

	const alphas = range(0,18).map(v => +(v/12).toFixed(2)).concat([1, 2, 5, Infinity])

	onMount(async () => {
		test_elem_1  = await fetch(`https://raw.githubusercontent.com/jstonge/allotaxp/main/src/routes/data/boys1895.json`).then((r) => r.json());
		test_elem_2  = await fetch(`https://raw.githubusercontent.com/jstonge/allotaxp/main/src/routes/data/boys1930.json`).then((r) => r.json());
	});

	
	// Title stuff
	// TODO: math is not reactive yet.
	$: math = `$$D_{\\alpha}^R (\\Omega_1 || \\Omega_2 = 0.234)$$\n$$\\propto \\sum_\\tau | \\frac{1}{r_{\\tau,1}^{alpha}} - \\frac{1}{r_{\\tau,2}^{0.92}} |$$`;
	$: title = ['Boys 1895', 'Boys 1930']

	// DATA WRANGLING - needs to be reactive, but could be refactor
	$: alpha = 0.92;
	$: me = combElems(test_elem_1, test_elem_2);
	$: rtd = RTD(me, alpha);
	$: dat = myDiamond(me, rtd);
	$: diamond_dat_raw = dat.counts;

	// Here because we need `diamond_dat_raw`. Could be better.
	$: max_rank_raw = range(max(diamond_dat_raw, d=>d.x1))
	$: relevant_types = get_relevant_types(diamond_dat)

	// Data for the plots
	$: diamond_dat = diamond_dat_raw.filter(d => d.types !== "")
	$: barData = wordShift_dat(me, dat).slice(0, 30);

	async function load_sys1(e) {
		title[0] = e.target.__value.name
		Papa.parse(e.target.__value, {
			header: true, 
			complete: function(results) {
				test_elem_1 = results.data;
				
			}
		});
	}

	async function load_sys2(e) {
		title[1] = e.target.__value.name
		Papa.parse(e.target.__value, {
			header: true, 
			complete: function(results) {
				test_elem_2 = results.data;
			}
		});
	}
    
    $: innerHeight = DiamondHeight - margin.inner;   
    $: diamondHeight = innerHeight - margin.diamond;

    $: ncells = d3.max(max_rank_raw)
	$: max_rank = d3.max(diamond_dat, (d) => d.rank_L[1]);
    $: rounded_max_rank = 10**Math.ceil(Math.max(Math.log10(max_rank)))
    $: xyDomain = [1, rounded_max_rank];

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

<svelte:head>
	<title>Home</title>
	<meta name="description" content="Svelte demo app" />
</svelte:head>


<aside>	
	<div class="alpha-slider">
		<p>Choose alpha:</p>
		<!-- here the value are the index of our alphas array, not the actual values-->
		<input 
			type="range"
			min="0"
			max="{alphas.length-1}"
			on:input={(e) => {alpha = alphas[e.target.value]}} 
			list=mapsettings
			>
		{#each alphas as ax}
			<datalist id="mapsetting"> <option>{ax}</option> </datalist>
		{/each}
		<p>α={alpha}</p>
	</div>
	<div>
		<p>Rank turbulence: </p>
		<MathJax {math}/>
	</div>
	<!-- Upload files from users without reading them -->
	<p>Upload files:</p>
	<!-- <label for="system1" class="custom-file-upload">Upload files..</label> -->
	<input type="file" id="system1" multiple name="system1" accept=".csv" bind:files>
	<!-- Create select for both systems, which we are gonna read on:change() -->	
	{#if files.length > 1}
			<p>select system 1</p>
			<div class="wrapper">
				<select size="{files.length}" bind:value={selected_sys1} on:click={load_sys1}>
					{#each files as file}
						<option value={file}>{file.name}</option>
					{/each}
				</select>		
			</div>
			<p>select system 2</p>
			<div class="wrapper">
				<select size="{files.length}" bind:value={selected_sys2} on:click={load_sys2}>
					{#each files as file}
						<option value={file}>{file.name}</option>
					{/each}
				</select>		
			</div>
	{/if}
	<p>
		<i>p.s. right now we are expecting `.csv` files with 4 columns, e.g. </i>
		<br><br>
		<code>
			types,counts,probs,total_unique
			John,9829,0.0653,1506
			William,8579,0.0570,1506
			James,7245,0.0481,150
			<br>...<br>
			Young,8,5.316e-5,1506
		</code>
		<br><br>
		<i>see <a href="https://github.com/jstonge/allotaxonometer/">jstonge/allotaxonometer</a> for more details.</i>
	</p>
</aside>
<main>
	<div class="dashboard" bind:clientWidth={DashboardWidth}>
		<!-- svelte-ignore a11y-no-static-element-interactions -->
		<svg id="mysvg" height={DashboardHeight} width={DashboardWidth} on:mouseleave={() => hoveredData = null}>
			<g>
				{#each title as suptitle, i}
					<text
						x={i == 0 ? 100 : 510}
						y={80}
						font-size="25px"
					>{suptitle}</text>
				{/each}
			</g>
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
						on:mouseover={() => { hoveredData = d }}
						on:focus={() => { hoveredData = d }}
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
			<Wordshift {barData} {DashboardHeight} {DashboardWidth}/>
			<DivergingBarChart {test_elem_1} {test_elem_2} {DiamondHeight} {DiamondWidth} /> 
			<Legend {diamond_dat} {DiamondHeight}/> 
		</svg>
		<!-- {#if hoveredData} 
			<Tooltip 
				data={hoveredData} width="200px" scale={xy}/>
		{/if} -->
	</div>
	<div class="download-button">
		<button on:click={downloadChart}>Download Image</button>
	</div>
</main>


<style>
	main {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		text-align: center;
		margin: 0 auto;
		margin-left: 150px;
		width: 100%;
	}

	@media (min-width: 800px) {


	.dashboard {
    	position: relative;
  	}
	aside {
			align-self: start;
			height: 100%; /* Full-height: remove this if you want "auto" height */
			width: 250px; /* Set the width of the sidebar */
			position: fixed; /* Fixed Sidebar (stay in place on scroll) */
			z-index: 1; /* Stay on top */
			top: 0; /* Stay at the top */
			left: 0;
			background-color: #ffffff83; /* Black */
			overflow-x: hidden; /* Disable horizontal scroll */
			padding-top: 20px;
			padding-left: 5px;
		}
	}


	.download-button {
		display: flex;
		flex-direction: row-reverse;
		margin-right: -900px;
		margin-top: 20px;
	}

	.wrapper{
		display: flex;
		align-items: center;
		justify-content: center;
		padding-top: 10px;
		max-height: 120px;
   		overflow-y: auto;
	}



</style>
