<script>
// @ts-nocheck
    import { fly } from "svelte/transition";

    export let data;
    export let scale;
    export let width;

    $: x = scale(data.x1);
    $: y = scale(data.y1);
  
    let tooltipWidth;
  
    const xNudge = 15;
    const yNudge = 30;
  
    $: xPosition = x + tooltipWidth > width ? x - tooltipWidth - xNudge : x + xNudge;
    $: yPosition = y + yNudge;

    // $: console.log((xPosition, yPosition))
</script>

 
<div
  class="tooltip"
  transition:fly={{ y: 10 }}
  bind:clientWidth={tooltipWidth}
  >
  <h1>Types: <span>{data.types}</span></h1>
</div>
  
<style>
    .tooltip {
      padding: 8px 6px;
      background: white;
      box-shadow: rgba(0, 0, 0, 0.15) 2px 3px 8px;
      border-radius: 3px;
      pointer-events: none;
      transition: top 300ms ease, left 300ms ease;
      position: absolute; 
      left: 0; 
      top: 0; 
    }
  
    h1,
  
    h1 {
      font-size: 1rem;
      font-weight: 400;
      width: 100%;
    }

    span {
      background: #dda0dd50;
      font-size: 80%;
      margin-left: 2px;
      padding: 2px 4px;
      display: inline-block;
      vertical-align: bottom;
      border-radius: 3px;
      color: rgba(0, 0, 0, 0.7);
    }
  </style>