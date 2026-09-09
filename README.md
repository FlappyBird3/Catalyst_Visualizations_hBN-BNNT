# Engineering hBN/BNNT Supported TMOx Nanocatalysts

Presented at the 2025 MRS Fall Meeting, Boston, MA — abstract EN01.10.28.
Krish Patel¹, Md Ashraful Islam², Tae Jin Kim², Miriam Rafailovich²
¹The Morris County Academy for Math, Science, and Engineering ²Stony Brook University, The State University of New York

---

## Abstract

Catalytic oxidation of carbon monoxide (CO) is a key reaction for air pollution control, exhaust purification, and indoor CO poisoning mitigation. Therefore, developing stable, efficient, and cost-effective catalysts for this reaction is crucial. While noble metals such as Pd and Pt traditionally offer the greatest catalytic activity, their high cost limits widespread application. However, transition metal oxides (TMOx) have emerged as promising catalysts for CO oxidation due to their lower cost and demonstrated catalytic activity at moderate temperatures; still, a strong support material is required to facilitate its catalytic activity. Tangentially, hexagonal boron nitride (hBN) and boron nitride nanotubes (BNNT) are thermally stable and chemically inert, making them a promising support material. With a 2D wafer structure similar to graphene, hBN enables a high dispersion of metal nanoparticles on its edges, reducing agglomeration. Structurally similar to 1D carbon nanotubes (CNTs), BNNT are more resistant to oxidation and offer less structural degradation at high temperatures.

One 1 wt% CuOx/hBN, one 1 wt% CuOx/BNNT(80), one 3 wt% CoOx/hBN, and one 3 wt% CoOx/BNNT catalysts were developed with the novel one-pot chemical vapor deposition method (OP-CVD), which includes a pre-mixing step to ensure homogeneity of metal precursor and support, evaporation to disperse the TMOx precursor, and calcination fixation step, promoting the oxidation of Cu/Co on the hBN/BNNT support. For this synthesis the organometallic copper precursor Cu(acac)2 and the cobalt precursor Co(acac)2 were used; hBN and BNNT(80) were chosen as BN support. The catalytic activity tests of 40 mg of the four CoOx/hBN, CoOx/BNNT, CuOx/hBN, and CuOx/hBN catalysts were performed in a fixed-bed flow reactor, which regulated appropriate flow rates for the carrier gas He, reactant gases O2 and 10% concentrated CO and product gas CO2. To analyze the support effect (hBN vs BNNT) and TMOx effect (CuOx vs CoOx) on catalytic activity, a temperature vs CO% curve was produced for each catalyst sample based on appropriate gas chromatography readings. Comprehensive characterization methods such as Transmission Electron Microscopy (TEM) and Fourier Transform Infrared Spectroscopy (FTIR) were applied to illustrate physicochemical properties and validate catalytic activity.

The TEM confirmed that the CoOx/hBN sample exhibited the least agglomeration with the greatest dispersion at the edge of hBN wafers, and FTIR of CuOx/hBN validated each step of the OP-CVD methodology. From the gas chromatography readings, CoOx nanocatalysts showed promising results: CoOx/BNNT exhibited T90 (90% conversion) as early as 205°C and CoOx/hBN by 275°C, while CuOx/hBN and CuOx/BNNT demonstrated <20% conversion up to 450°C. This study validated a method to produce catalytic synthesis of transition metal oxides at 1 and 3 wt% for both hBN and BNNT and affirmed cobalt oxide's greater synergistic effect with hBN/BNNT as compared to copper oxide; the study also identified CoOx's greater CO oxidation catalytic performance with BNNT support compared to hBN.

---

## This repository shows computational visualizations

`CuOx_hBN_Simulation.ipynb` builds three-dimensional atomic models of two of the four catalysts described above: **1 wt% CuOx/hBN** and **1 wt% CuOx/BNNT(80)**. The models are constructed with ASE (the Atomic Simulation Environment, a Python library for building and handling atomic structures) and displayed with NGLView, which draws atoms as spheres and bonds as connecting rods inside the notebook.

The purpose of these models is to show, at the level of individual atoms, what a 1 wt% loading of copper oxide on a boron nitride support actually looks like. The physical samples cannot be observed this way. TEM shows dark regions where copper has collected, but it does not show individual atoms or their arrangement. The models fill in that missing picture, and they also make the loading concrete: they answer the question of how many copper oxide units 1 wt% actually corresponds to.

The notebook contains four code cells. The first three produce structure visualizations. The fourth attempts a machine learning analysis and does not currently run.

---

## Visualization 1 — Copper oxide on a small sheet of hBN

The code builds a flat sheet of hexagonal boron nitride by repeating a two-atom pattern (one boron, one nitrogen) six times in each direction, giving 72 atoms. It then adds 15 Å of empty space above and below the sheet so that the sheet does not interact with copies of itself. An Ångström (Å) is one ten-billionth of a meter; a boron–nitrogen bond is about 1.45 Å long.

The display shows a honeycomb sheet of alternating boron and nitrogen atoms with copper oxide molecules positioned above it.

**However, the number of copper oxide molecules placed is zero.** This is worth understanding, because it is a real physical result rather than a programming mistake:

- The 72-atom sheet has a mass of 893 atomic mass units.
- One percent of that is 8.9 units.
- One CuO molecule weighs 79.5 units.
- 8.9 divided by 79.5 is 0.11, which rounds down to zero.

In other words, **a 72-atom sheet is too small to hold a 1 wt% loading.** Adding even one copper oxide molecule would produce 8.2 wt%, more than eight times the experimental loading. Reaching 1 wt% requires a support of at least a few thousand atoms. This is the reason the second cell exists.

---

## Visualization 2 — Copper oxide on a large sheet of hBN

This cell reverses the logic. Instead of choosing a sheet size and calculating how much copper it implies, it sets a target support mass and calculates the sheet size needed.

- The sheet is expanded to 43 × 43 repeats, giving **3,698 atoms** and a mass of 45,887 atomic mass units.
- One percent of that mass is 459 units, which corresponds to **6 CuO molecules**.
- The sheet measures 107.5 × 93.1 Å, an area of about 100 square nanometers.

The display shows a large honeycomb sheet with six isolated copper oxide molecules scattered across it at random positions. Because six molecules on a sheet roughly 11 nanometers across is a very small amount of material, the visualization looks mostly empty, and the copper must be located by rotating and zooming.

**That emptiness is the scientific point.** It shows directly what 1 wt% means: the copper is a trace addition, and if it is well dispersed, individual copper oxide units sit far apart from one another with large stretches of bare support between them.

**One correction is needed.** The code intends to place the copper 3 Å above the sheet, but the number `3.0` is measured from the bottom of the simulation box rather than from the sheet itself. Because the sheet was moved to a height of 15 Å, the copper ends up 12 Å **below** it, floating in empty space with no contact with the support. The first cell handles this correctly, so the intent is clear elsewhere in the notebook. The structure should be regenerated before it is used for anything further.

---

## Visualization 3 — Copper oxide inside a boron nitride nanotube

ASE has no built-in nanotube builder for boron nitride, so the code builds a carbon nanotube first and then relabels every atom as either boron or nitrogen, alternating between the two.

I checked whether this relabeling produces a chemically valid structure by counting every bond in the tube. **Every boron is bonded only to nitrogen and every nitrogen only to boron**, with no boron–boron or nitrogen–nitrogen contacts anywhere. This is the correct arrangement for boron nitride, so the shortcut works.

The resulting tube:

- Contains **3,600 atoms** and is **22.1 nanometers long** and **1.36 nanometers wide**.
- Holds **6 CuO molecules** at 1 wt%, the same count as the flat sheet.
- Places those molecules inside the tube, about 3 Å from the inner wall.

The display shows a long, narrow tube of alternating boron and nitrogen atoms with copper oxide molecules inside it. Viewing the copper requires rotating to look down the length of the tube.

**Two modeling choices are worth reconsidering:**

- **The copper is placed inside the tube, not on the outside.** In the OP-CVD synthesis, the copper precursor is mixed with the support, evaporated, and heated. Nanotubes as produced are usually closed at their ends, so the precursor is unlikely to get inside. Deposition on the outer surface is the more likely outcome and is what TEM would show. Building an outer-surface version and comparing the two would make the model match the experiment more closely.
- **The copper occupies only the middle third of the tube.** The code treats a length setting as though it were measured in Ångströms when it actually counts repeat units, so all six molecules land within an 80 Å window of a 221 Å tube.

---

## How the visualizations connect to the experimental results

**They give a reference point for reading the TEM images.** The abstract reports that CoOx/hBN showed the least agglomeration and the best dispersion at the edges of hBN wafers. Agglomeration means the oxide has clumped into larger particles; dispersion means it has stayed spread out. The models represent the ideal limit of perfect dispersion, since six separate molecules are placed and nothing allows them to merge.

That limit converts into a useful comparison for the TEM:

- A **1 nm** copper oxide particle contains about **25** CuO units.
- A **2 nm** particle contains about **200**.
- A **5 nm** particle contains about **3,100**.

The entire 1 wt% copper content of the 100 nm² modeled sheet is 6 units — less than a quarter of a single 1 nm particle. Any visible particle in a TEM image therefore draws copper from a support area far larger than the simulation box. This provides a way to translate an observed particle size into an estimate of how much of the copper has agglomerated.

**They do not yet represent the edges, which is where the key TEM observation lies.** The modeled sheet is treated as repeating infinitely in both flat directions, which means it has no boundary at all. Since the abstract identifies edges as the site of greatest dispersion, a follow-up model would need a finite flake or a strip with defined edges to study that effect.

---

## Running the notebook

```bash
pip install ase nglview matplotlib
```

NGLView requires a live Jupyter session. It does not display on GitHub or in exported HTML, so the cells must be re-run to see the structures. For images intended for a poster or paper, open the saved structure files in VESTA, OVITO or VMD, which produce higher-quality renderings.

### Files produced

| File | Contents |
|---|---|
| `cuo_on_hbn.xyz` / `.vasp` | Small hBN sheet, 72 atoms, no copper |
| `cuo_on_hbn_random_above_B_and_N_surface_fully_randomized.xyz` / `.vasp` | Large hBN sheet, 3,698 atoms, 6 CuO |
| `cuo_inside_bnnt.xyz` / `.vasp` | Nanotube, 3,600 atoms, 6 CuO |

The `.xyz` files are for viewing. The `.vasp` files are inputs for quantum-mechanical calculations. In both cases the atoms are placed at estimated positions and have not been allowed to settle into their true arrangement, so a relaxation step is required before any calculated energy is meaningful.

### Verified numbers

| | Sheet (small) | Sheet (large) | Nanotube |
|---|---|---|---|
| Support atoms | 72 | 3,698 | 3,600 |
| Support mass (u) | 893 | 45,887 | 44,671 |
| CuO molecules at 1 wt% | **0** | 6 | 6 |
| Support area (nm²) | 1.9 | 100.1 | 94.3 |
| Copper position | 2.5 Å above sheet | 12 Å **below** sheet | inside tube |

All values were obtained by re-running the notebook's geometry code, not estimated.
