<script>
  // @ts-nocheck
  import * as d3 from "d3";

  export let visHeight;
  export let alpha;

  function rank_turbulence(x, y, alpha) {
    if (alpha === Infinity) {
      return x == y ? 0 : Math.max(x);
    } else if (alpha == 0) {
      return (1 + 1) / 1 * Math.abs(x ** 1 - y ** 1) ** (1 / (1 + 1));
    } else {
      return (alpha + 1) / alpha * Math.abs(x ** alpha - y ** alpha) ** (1 / (alpha + 1));
    }
  }

  // Parameters for grid
  const q = 4; 
  const x0 = -1;
  const y0 = -1;

  function createGrid(visHeight, alpha) {
    const x1 = visHeight + q - 2;
    const y1 = visHeight + q - 2;
    const n = Math.ceil((x1 - x0) / q);
    const m = Math.ceil((y1 - y0) / q);
    const gridData = Array.from({ length: n * m }, (_, idx) => {
      const i = idx % n;
      const j = Math.floor(idx / n);
      return rank_turbulence(i, j, alpha);
    });

    return { data: gridData.reverse(), x: -q, y: -q, k: q, n, m };
  }

  // Reactive grid based on visHeight and alpha
  $: grid = createGrid(visHeight, alpha);

  // Convert grid coordinates to screen coordinates
  const transform = ({ type, value, coordinates }) => ({
    type,
    value,
    coordinates: coordinates.map(rings =>
      rings.map(points =>
        points.map(([x, y]) => [
          grid.x + grid.k * x + 3,
          grid.y + grid.k * y + 3
        ])
      )
    )
  });

  // Generate contours
  $: contour = d3.contours()
                 .size([grid.n, grid.m])
                 .thresholds(10)
               (grid.data)
                 .map(transform);

  let mypath = d3.geoPath();
</script>

<g class="contours">
  {#each contour as line, index}
    <g>
      <path 
        d={mypath(line)} 
        fill="none"
        stroke="grey" 
        opacity={1}
      ></path>
    </g>
  {/each}
</g>
