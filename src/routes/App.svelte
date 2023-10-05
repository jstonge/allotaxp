<script>
  import { test_elem_1, test_elem_2 } from './data/data.js'
  import { combElems, RTD, myDiamond, wordShift_dat } from "./components/combine_distributions.js"

  import Diamond from "./components/Diamond.svelte";
  import Wordshift from "./components/BarChart.svelte";

  import Scrolly from "./helpers/Scrolly.svelte"

  let alpha = 0.92;
  
  const me = combElems(test_elem_1, test_elem_2)
  const rtd = RTD(me, alpha)
  const dat = myDiamond(me, rtd)
  const diamond_dat_raw = dat.counts
  const diamond_dat = diamond_dat_raw.filter(d => d.types !== "")
  
  const me2 = combElems(test_elem_2, test_elem_1)
  const rtd2 = RTD(me2, alpha)
  const dat2 = myDiamond(me2, rtd2)
  const diamond_dat_raw2 = dat2.counts
  const diamond_dat2 = diamond_dat_raw2.filter(d => d.types !== "")
  
  const me3 = combElems(test_elem_1, test_elem_2)
  const rtd3 = RTD(me3, 0)
  const dat3 = myDiamond(me3, rtd3)

  const barData = wordShift_dat(me, dat).slice(0, 30);
  const barData2 = wordShift_dat(me3, dat3).slice(0, 30);

  let currentStep;

  let initialData = diamond_dat;
  let renderedData = initialData;
  
  let initialBarData = barData;
  let rendererBarData = initialBarData;

  // Series of if-else to play with data.
  $: {
    if (currentStep === 0) {
      renderedData = initialData.map((d => ({...d, x1: 0, y1: 0})));
      rendererBarData = initialBarData.map((d => ({...d, x1: 0, y1: 0})));
    } else if (currentStep === 1) {
      renderedData = initialData;
      rendererBarData = initialBarData;
    } else if (currentStep === 2) {
      renderedData = diamond_dat2;
      rendererBarData = initialBarData;
      console.log(rendererBarData)
    } else if (currentStep === 3) {
      renderedData = initialData;
      rendererBarData = barData2;
      console.log(rendererBarData)
    }
  }
</script>

<main>
  <h1>Allotaxonometry for all</h1>
  <h2>Boys 1895 vs 1930</h2>
  <section>
    <div class='sticky'>
      <div style="display:flex; gap: 3em;">
        <div> <Diamond diamond_dat={renderedData}/> </div>
        <div> <Wordshift barData={rendererBarData} /> </div>
      </div>
    </div>
    
    <div class="steps">
      <Scrolly bind:value={currentStep}>
        {#each ["Scrollytelling", "♥", "Allotaxonometry", "4 All"] as text, i}
        <div class='step' class:active={currentStep === i}>
          <div class="step-content">
            <p>{text}</p>
          </div>
        </div>
        {/each}
      </Scrolly>
    </div>
    <div class='reference-step'>
      {currentStep}
    </div>
  </section>
</main>


<style>
  main {
    text-align: center;
    margin: 0 auto;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    min-height: 120vh;
    background: white;
  }

  h1 {
    font-size: 50px;
  }

  :global(rect) {
    transition: 
      fill 700ms cubic-bezier(0.76, 0, 0.24, 1),
      x 700ms cubic-bezier(0.76, 0, 0.24, 1), 
      width 700ms cubic-bezier(0.76, 0, 0.24, 1),
      y 700ms cubic-bezier(0.76, 0, 0.24, 1);
  }

  :global(text) {
    transition: 
      x 700ms cubic-bezier(0.76, 0, 0.24, 1), 
      y 700ms cubic-bezier(0.76, 0, 0.24, 1);
  }


  .step {
    height: 90vh;
    opacity: 0.3;
    transition: opacity 300ms ease;

    display: flex;
    justify-content: center;
    place-items: center;
  }

  .step.active {
    opacity: 1;
  }

  .step-content {
    background-color: white;
    border: 1px solid black;
    padding: 0.75rem 1rem;
    border-radius: 3px; 
  }

  section {
    position: relative;
  }

  .steps {
    position: relative;
    z-index: 2;
  }

  .sticky {
    position: sticky;
    top: 0;
    z-index: 1;
  }

  .reference-step {
    position: fixed;
    bottom: 0;
    right: 0;
    padding: 1rem;
  }
</style>
