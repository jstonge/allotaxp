<script>
    import Diamond from "./Diamond.svelte";
    import Wordshift from "./BarChart.svelte";
    import mixedElems from "./combine_distributions.js"; 
    import { tweened } from "svelte/motion";
    
    export let diamond_dat;
    export let barData;

  
  </script>
  
  <div style="display:flex; ">
    <div>
      <Diamond {diamond_dat}/>
    </div>
    <div>
      <Wordshift {barData}/>
    </div>
  </div>
