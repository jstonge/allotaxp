# ALLotaxonometer

<p align="center">
  <img width="600" alt="Screenshot 2023-10-05 at 9 39 50 AM" src="https://github.com/jstonge/allotaxp/assets/35715881/8b5c4a10-18eb-4c21-bbb1-b736a3886273">
</p>

- See [jstonge/allotaxonometer](https://github.com/jstonge/allotaxonometer) for wrangling the input data in the right format.
- Another version lives at [https://observablehq.com/@jstonge/allotaxonometer-4-all](https://observablehq.com/@jstonge/allotaxonometer-4-all)

### Current features

 - provide `.csv` files to make your analysis
 - reactive slider to change the `alpha` parameter
 - scrollytelling adventure to learn the `allotaxonometer` (still beta)
 - download the output (.png at the moment)

### Roadmap

 - [ ] fix hover; right now hover doesn't see the transformation of the `rect` positions.
 - [ ] ensure the dashboard looks well across platforms and screen sizes.
 - [ ] add contour lines:
   - [ ] $(\frac{1}{r_{1}^{\alpha}} - \frac{1}{r_{2}^{\alpha}}) = c$
   - [ ] $\frac{1}{r_{1}^{\alpha}} = c + \frac{1}{r_{2}^{\alpha}}$
   - [ ] $r_1 = (c + \frac{1}{r_{2}})^{-\alpha}$
 - [ ] make `alpha` slider reactive;
 - [ ] integrate the `alph` slider to the `svg`
 - [ ] add metrics beyond `rank_turbulence_divergence`
 - [ ] call the `allotaxonometer.js` package instead of having the helper files in the project
 - [ ] the output should be a `.pdf`
 - [ ] python version to use the allotax programmatically
