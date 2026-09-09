# CuOx_hBN_Simulation.ipynb

Atomistic structure models of 1 wt% copper oxide supported on hexagonal boron nitride (hBN) and on a boron nitride nanotube (BNNT), built with the Atomic Simulation Environment (ASE) and displayed with NGLView.

These models correspond to two of the four catalysts in the MRS Fall 2025 poster EN01.10.28, *Engineering hBN/BNNT Supported TMOx Nanocatalysts — Support and TMOx Effect on Catalytic Performance for CO Oxidation* (Patel, Islam, Kim, Rafailovich): specifically the **1 wt% CuOx/hBN** and **1 wt% CuOx/BNNT(80)** samples.

---

## 1. Vocabulary used in this README

Terms are defined here once so the rest of the document can use them without interruption.

- **Ångström (Å)** — a unit of length equal to 10⁻¹⁰ m, or 0.1 nanometers. A boron–nitrogen bond is about 1.45 Å.
- **Atomic mass unit (u)** — a unit of mass equal to 1/12 the mass of a carbon-12 atom. One CuO formula unit has a mass of 79.545 u.
- **Unit cell** — the smallest repeating group of atoms that, when copied in a regular pattern, reproduces the whole crystal. The hBN unit cell used here holds one boron atom and one nitrogen atom.
- **Lattice constant** — the edge length of the unit cell. The notebook uses `a = 2.5 Å` for hBN.
- **Supercell** — a block made by copying the unit cell a whole number of times in each direction. A 6×6 supercell of hBN contains 36 unit cells and therefore 72 atoms.
- **Periodic boundary conditions (PBC)** — a modeling convention in which the supercell is treated as repeating infinitely. An atom that exits one face of the box re-enters through the opposite face. The practical consequence is that a structure with PBC turned on in a given direction has **no edges** in that direction.
- **Vacuum padding** — empty space added along a direction where periodicity is not wanted. Adding 15 Å of vacuum above and below a two-dimensional sheet separates each copy of the sheet from the copy above it far enough that they do not interact.
- **Ball-and-stick representation** — a display convention in which atoms are drawn as spheres and bonds as cylinders. NGLView infers which atoms to connect by distance; it does not perform any chemistry.
- **Chirality indices (n, m)** — two integers that fully specify how a nanotube is rolled from a flat sheet. They determine the tube's diameter and its symmetry. A tube with n = m is called *armchair*; the notebook uses (10, 10).
- **XYZ file** — a plain-text file listing each atom's element symbol and its x, y, z coordinates. It is the most portable structure format and is used mainly for viewing.
- **POSCAR / `.vasp` file** — the structure input format for the VASP density functional theory code. It stores the three cell vectors and the atomic positions, and is what you would hand to an electronic-structure calculation.
- **Density functional theory (DFT)** — a quantum-mechanical method for computing the electronic energy of a set of atomic positions. It is the standard way to obtain adsorption energies and reaction barriers.
- **Geometry relaxation** — the procedure of moving atoms until the forces on them are near zero, which finds the nearest stable arrangement. None of the structures in this notebook have been relaxed.

---

## 2. What the notebook contains

Five cells. Four contain code; the fifth is empty.

| Cell | Purpose | Visualization produced | Status in the saved file |
|---|---|---|---|
| 1 | 1 wt% CuO on a small hBN sheet, molecules placed on an evenly spaced line | NGLView 3D widget | Raised `ModuleNotFoundError: No module named 'ase'` |
| 2 | 1 wt% CuO on a large hBN sheet, molecules placed at random positions | NGLView 3D widget | Ran; printed masses and displayed the widget |
| 3 | 1 wt% CuO inside a (10,10) BNNT | NGLView 3D widget | Ran; displayed the widget |
| 4 | Random forest regression on Materials Project data | Matplotlib scatter plot | Raised `ImportError: cannot import name 'MPRester'` |
| 5 | — | — | Empty |

The saved widget outputs are `NGLWidget()` placeholders. NGLView renders through a Jupyter widget, and the rendered image is not stored inside the `.ipynb` file. The cells must be re-executed in a live kernel for the structures to appear.

---

## 3. Cell 1 — CuO on a small hBN sheet, evenly spaced

### What the code builds

- A two-atom hBN unit cell is defined with boron at fractional coordinate (0, 0, 0) and nitrogen at (1/3, 2/3, 0). Fractional coordinates express position as a fraction of each cell vector rather than in Å.
- The cell vectors describe a hexagonal lattice: **a₁** = (2.5, 0, 0), **a₂** = (−1.25, 2.165, 0), **a₃** = (0, 0, 10). The 120° angle between **a₁** and **a₂** is what produces the honeycomb arrangement.
- Periodicity is set to `[1, 1, 0]`, meaning the sheet repeats infinitely in x and y but not in z.
- The cell is copied 6 times in x and 6 times in y, giving 36 unit cells and **72 atoms**.
- 15 Å of vacuum is added above and below, producing a box measuring 15.00 × 12.99 × 30.00 Å with the sheet lying at z = 15.00 Å.
- A CuO molecule is defined as two atoms separated by 1.73 Å along z. This is close to the measured bond length of the gas-phase CuO diatomic molecule, about 1.724 Å, so the choice is physically reasonable for an isolated molecular unit.
- A rotation matrix representing a 90° rotation about the x axis is applied, which reorients the Cu–O axis from +z to −y. The molecule therefore lies **parallel to the sheet**, not perpendicular to it.

### What the visualization shows

A flat hexagonal sheet of alternating boron and nitrogen atoms, viewed in ball-and-stick form, with a row of CuO molecules positioned 2.5 Å above the sheet and spaced evenly along the x direction at the midpoint of y.

### The number of CuO molecules is zero

This is the most important behavior in this cell. The weight-percent calculation is:

```
mass of hBN supercell         = 893.41 u
target CuO mass (1 wt%)       = 8.93 u
mass of one CuO formula unit  = 79.55 u
number of CuO units           = round(8.93 / 79.55) = round(0.112) = 0
```

A 6×6 supercell is too small to represent a 1 wt% loading with a whole number of CuO units. One CuO molecule on 72 hBN atoms would be 8.2 wt%, which is more than eight times the experimental loading. Because `int(round(...))` rounds 0.112 down to zero, the placement loop never executes and the saved structure contains hBN only.

This is not a coding error so much as a physical constraint: **at 1 wt%, the supercell must contain at least a few thousand support atoms before a single CuO unit is the correct amount.** Cell 2 exists to address exactly this, which is why it begins by specifying a target support mass instead of a supercell size.

---

## 4. Cell 2 — CuO on a large hBN sheet, randomly placed

### What the code builds

This cell inverts the logic of Cell 1. Rather than choosing a supercell and computing how much CuO it implies, it chooses a support mass and computes the supercell needed.

- A target boron-plus-nitrogen mass of 48,000 u is set.
- Dividing by the 24.82 u mass of one B–N unit cell gives 1,934 unit cells required.
- The supercell repeat count is taken as `int(sqrt(1934))` = **43**, so the sheet is 43 × 43 = 1,849 unit cells and **3,698 atoms**.
- The realized support mass is 45,886.63 u, which is 4.4% below the 48,000 u target. The shortfall comes from truncating √1934 = 43.98 down to 43. This does not introduce an error in the loading, because the weight percent is computed from the realized mass, not the target.
- 1 wt% of 45,886.63 u is 458.87 u, which divided by 79.545 u gives 5.77, rounded to **6 CuO molecules**.
- The final box measures 107.50 × 93.10 × 30.00 Å. The sheet occupies 107.50 × 93.10 Å = 100.1 nm².

Six CuO units over 100.1 nm² is an areal density of **0.060 CuO formula units per nm²**.

### What the visualization shows

A large hBN sheet with six isolated CuO molecules scattered across it at random lateral positions. Because there are only six molecules on a sheet nearly 11 nm across, the widget will appear mostly empty, and the CuO units must be located by rotating and zooming.

### Two issues to correct before using this structure

**Issue A — the CuO molecules are below the sheet, not above it.**

The variable `z_offset = 3.0` is an absolute coordinate, but the sheet was moved to z = 15.00 Å by `hbn.center(vacuum=15.0, axis=2)`. Placing the CuO at z = 3.0 puts it **12 Å below the plane of the sheet**, inside the lower vacuum region, with no contact with the support. The comment in the code states "3 Å above the surface," which is the intent, but the arithmetic does not implement it.

The fix is to make the offset relative to the sheet:

```python
z_sheet = hbn.get_positions()[:, 2].mean()   # 15.0 Å
z_offset = z_sheet + 3.0                      # 18.0 Å
```

Cell 1 does this correctly with `z_offset = cell_z / 2 + 2.5`, so the intent is clearly established elsewhere in the notebook.

**Issue B — the boron and nitrogen site positions are computed but never used.**

The code extracts boron positions with `hbn.get_positions()[::2]` and nitrogen positions with `[1::2]`. This indexing is valid: ASE's supercell construction tiles the two-atom basis, so even indices are boron and odd indices are nitrogen throughout the 3,698-atom sheet. I confirmed this on the actual structure.

However, the arrays are combined, shuffled, and then iterated over without being read. Inside the loop, placement uses `random.uniform(0, cell_x)` and `random.uniform(0, cell_y)` instead of `pos`. The CuO molecules therefore land at arbitrary points in the plane rather than above identified B or N sites. If site-specific adsorption is the goal — and it should be, since boron and nitrogen sites bind metal atoms differently — the placement line should read:

```python
cuo_i.translate([pos[0], pos[1], z_offset])
```

with `all_positions` restricted to whichever sublattice you intend to study.

---

## 5. Cell 3 — CuO inside a BNNT

### What the code builds

- ASE has no direct boron nitride nanotube builder, so a carbon nanotube is constructed first and its atoms are then relabeled. Chirality indices (10, 10) and `length=90` are used.
- **`length` counts unit cells, not Ångströms.** The (10,10) armchair unit cell contains 40 atoms and is 2.46 Å long, so the tube has **3,600 atoms** and an axial extent of **221.36 Å, or 22.1 nm**.
- Atoms are relabeled by index parity: even indices become boron, odd indices become nitrogen. I checked whether this produces a chemically correct structure by counting nearest-neighbor pairs, and it does: **0% of the 5,371 bonds are boron–boron or nitrogen–nitrogen**. Every boron is bonded only to nitrogen and every nitrogen only to boron, which is the correct alternating sublattice arrangement for boron nitride. This also holds for the other chiralities I tested, including (10,0), (8,4), (6,3) and (9,6), so the approach is generalizable.
- The tube diameter is **13.56 Å**, or 1.356 nm.
- Support mass is 44,670.60 u, 1 wt% of which is 446.71 u, giving **6 CuO molecules** again.
- Each CuO is placed at a random azimuthal angle and a random axial position on a circle of radius 3.78 Å from the tube axis. Since the wall is at radius 6.78 Å, each molecule is about **3.0 Å from the inner wall**.

### What the visualization shows

A long single-walled tube of alternating boron and nitrogen atoms with six CuO molecules positioned inside the lumen, near the wall but not touching it. `nv_view.center()` re-centers the camera on the structure. Because the tube is 22.1 nm long and only 1.36 nm across, the default view is highly elongated, and the interior CuO molecules are difficult to see without rotating to look down the tube axis.

### Four modeling choices worth reconsidering

- **The CuO units occupy only the middle third of the tube.** The axial placement window is `center ± (length/2 − 5)`, which evaluates to ±40 Å because `length` is 90 unit cells and is being treated as though it were 90 Å. The result is that all six molecules fall within an 80 Å window, covering **36% of the 221 Å tube**. If uniform distribution along the full tube is intended, derive the bounds from the actual positions rather than from `length`.

- **The tube has open, unterminated ends.** `cnt.center(vacuum=20.0, axis=(0, 1, 2))` adds 20 Å of vacuum along all three axes, including the tube axis. This converts an infinite periodic tube into a finite segment whose end atoms are under-coordinated, meaning they have fewer bonding partners than the chemistry requires. In a DFT calculation, under-coordinated edge atoms produce electronic states that do not exist in a real closed or infinite tube, and these states can dominate the result. To keep the tube periodic along its axis, center only on the radial directions:

  ```python
  cnt.center(vacuum=20.0, axis=(0, 1))
  ```

- **The CuO molecules are placed inside the tube.** In the OP-CVD synthesis described in the abstract, the Cu(acac)₂ precursor is mixed with the support, evaporated, and calcined. As-produced BNNTs are typically capped at the ends, and precursor entry into the lumen requires open ends and favorable capillary conditions. Deposition on the **outer** surface is the more likely outcome and is what TEM would most readily show. An outer-surface variant is a one-line change: use a placement radius of `r_wall + 2.5` instead of `r_inner − 2.0`.

- **All six CuO molecules share one orientation.** The rotation matrix is applied to the CuO template once, before the placement loop, so every molecule has its Cu–O axis pointing along −y in the fixed laboratory frame. Because the molecules sit at different angles around the circumference, their orientation **relative to the local wall** differs from one to the next: some are approximately radial, others approximately tangential. If a consistent orientation with respect to the wall is intended, the rotation must be applied per molecule using its azimuthal angle.

- **The bond length is carried over from carbon.** ASE's nanotube builder uses a default bond length of 1.42 Å, appropriate for carbon–carbon bonds. The boron–nitrogen bond is about 1.45 Å. The tube built here is consequently about 2% narrower than a real (10,10) BNNT. Passing `bond=1.45` to `nanotube()` corrects this.

---

## 6. Cell 4 — Random forest regression and the actual-versus-predicted plot

### What the code intends

To query the Materials Project database for structural and electronic properties of CuO, hBN and BNNT, train a random forest regression model on those properties, and produce a scatter plot of actual against predicted energy.

A **random forest regressor** is a machine learning model that averages the predictions of many decision trees, each trained on a different random subset of the data. **R²** (the coefficient of determination) measures the fraction of variance in the target that the model explains; 1.0 is a perfect fit and 0.0 is no better than predicting the mean. **F1 score** is the harmonic mean of precision and recall for a two-class classification problem.

### Why the cell does not run, and why it would not be informative if it did

The immediate failure is an import error: `from pymatgen import MPRester` reflects an older package layout. The current import is `from mp_api.client import MPRester`, with an API key obtained from materialsproject.org.

Beyond the import, there are four substantive problems.

- **The query arguments are not valid identifiers.** `mpr.query({"material_ids": material}, ...)` is passed the strings `'CuO'`, `'hBN'` and `'BNNT'`. Materials Project IDs have the form `mp-704645`. Additionally, hBN is stored under the chemical formula `BN`, not `hBN`, and **boron nitride nanotubes are not in the database at all** — Materials Project contains three-dimensional periodic bulk crystals, and a nanotube is neither.

- **The target variable is also an input feature.** The features are `["energy", "band_gap", "formation_energy", "density"]` and the target is `df["energy"]`. The model is asked to predict a quantity it already receives. This is called **target leakage**, and it guarantees an R² near 1.0 that reflects nothing about the model's predictive ability. Removing `"energy"` from the feature list is required before any reported metric is meaningful.

- **The sample size cannot support the method.** Even with correct queries, three materials return on the order of a few entries. An 80/20 train-test split of such a set leaves one or two test points, and a 100-tree random forest has far more parameters than the data can constrain.

- **The properties are not descriptors of catalytic activity.** Total energy, band gap, formation energy and density of the bulk crystals do not predict CO oxidation performance on a supported nanocatalyst. Quantities that would be relevant include the CO adsorption energy on the CuOx unit, the oxygen vacancy formation energy in the oxide, the d-band center of the metal, and the binding energy between the oxide unit and the support.

There is also a smaller inconsistency: the comment on `new_material_data` lists the column order as `[band_gap, formation_energy, energy, density]`, but the scaler was fitted on `[energy, band_gap, formation_energy, density]`. The prediction would use mismatched columns.

### What the visualization would show

A scatter plot with true energy on the horizontal axis and predicted energy on the vertical axis. A well-performing model produces points close to the 45° diagonal. As currently written, target leakage would place every point exactly on that diagonal, and the plot would not be evidence of anything.

---

## 7. How these structures relate to the experimental work

### The loadings match the samples

The wt% arithmetic in Cells 2 and 3 reproduces the experimental 1 wt% loading for CuOx/hBN and CuOx/BNNT(80). The models are therefore loading-matched to the two copper samples, which makes them a valid starting point for structure-based interpretation of those specific results.

### The models represent perfect dispersion, which is the reference case for the TEM analysis

The poster reports that TEM showed CoOx/hBN with the least agglomeration and the greatest dispersion at the edges of hBN platelets, with the CuOx samples less favorable. **Agglomeration** means that individual oxide units have merged into larger particles; **dispersion** means they remain separated.

The models here place six isolated CuO formula units and provide no mechanism for them to combine. They therefore represent the limiting case of complete atomic dispersion. That limit is quantitatively useful for reading the TEM images:

- A 1 nm spherical CuO particle contains about **25** formula units.
- A 2 nm particle contains about **200**.
- A 5 nm particle contains about **3,100**.

Against those numbers, the entire 1 wt% copper content of the modeled 100 nm² sheet — six formula units — is less than a quarter of a single 1 nm particle. Any distinct CuOx particle visible in your TEM images at these length scales therefore represents copper drawn from a support area much larger than the simulation box. This gives a direct way to convert an observed particle size distribution into a statement about what fraction of the copper is agglomerated versus dispersed, which strengthens the TEM section of the poster.

### The hBN model has no edges, and edges are where the poster's key observation lies

Because periodicity is set to `[1, 1, 0]`, the hBN sheet repeats infinitely in the plane and has no boundary. The abstract specifically identifies **edge sites** as the location of greatest dispersion. The current model cannot represent that observation at all.

To study edge sites you would need one of the following:

- A **nanoribbon**: keep periodicity along one in-plane direction and add vacuum along the other, producing two parallel edges. The dangling bonds at the edges must be terminated, usually with hydrogen.
- A **finite flake**: set periodicity to `[0, 0, 0]` and add vacuum in all three directions, again with terminated edges.

Both are modest modifications to Cell 2 and would allow direct comparison of a CuO unit bound at an edge against one bound on the basal plane, which is the central structural question raised by the TEM result.

### The structures cannot yet explain the catalytic result

The poster reports that CuOx/hBN and CuOx/BNNT gave under 20% CO conversion up to 450 °C, while CoOx/BNNT reached 90% conversion (T90) at 205 °C and CoOx/hBN at 275 °C. That is a large difference between the copper and cobalt systems, and explaining it requires energies, not geometries.

The notebook produces `.xyz` and `.vasp` files but calls no energy method. Nothing in the notebook computes an adsorption energy, a reaction barrier, or a charge transfer. The files are inputs to such a calculation, not results of one.

For CO oxidation over reducible transition metal oxides, a common mechanism is **Mars–van Krevelen**, in which CO reacts with an oxygen atom taken from the oxide lattice, leaving an oxygen vacancy that is subsequently refilled from gas-phase O₂. The governing quantity is the **oxygen vacancy formation energy**: if it is too high, lattice oxygen is not available and the reaction is slow. This offers a testable explanation for the CuOx versus CoOx difference, and it is the natural next calculation.

It also identifies a limitation of the present models. A single CuO diatomic unit has one oxygen atom and no oxide lattice, so it cannot represent a vacancy mechanism at all. Modeling Mars–van Krevelen requires small **clusters** — Cu₄O₄, Cu₈O₈, or similar — rather than isolated formula units. Fixing this is not simply a matter of increasing the count; it requires building a cluster and placing it as one object.

---

## 8. Priority list for revision

Ordered by how much each change affects the scientific conclusions.

1. Correct `z_offset` in Cell 2 so the CuO molecules are above the sheet rather than 12 Å below it. The current structure has no metal–support contact and is not usable as a DFT input.
2. Remove `"energy"` from the feature list in Cell 4, or discard the machine learning cell entirely. As written it cannot produce an informative result.
3. Remove the axial vacuum in Cell 3 so the nanotube is periodic along its axis and has no unterminated ends.
4. Replace isolated CuO diatomic units with small CuₙOₙ clusters, so that oxygen vacancy chemistry can be represented.
5. Build an hBN nanoribbon or flake with terminated edges, so the edge dispersion reported in the TEM can be modeled.
6. Either use the extracted B and N site positions in Cell 2's placement loop, or delete them so the code states what it does.
7. Move the CuO from inside the BNNT to the outer surface, or build both and compare, to match the deposition geometry that OP-CVD most plausibly produces.
8. Pass `bond=1.45` to `nanotube()` for the correct boron–nitride bond length.
9. Set `np.random.seed()` and `random.seed()` in Cell 2 as is already done in Cell 3, so the randomized structures are reproducible.
10. Delete or repair Cell 1, which currently produces a bare hBN sheet with no copper.

---

## 9. Running the notebook

Cell 1 failed in the saved file because ASE was not installed in the `mrs` kernel at the time of execution.

```bash
pip install ase nglview matplotlib
jupyter nbextension enable nglview --py --sys-prefix   # classic Notebook only
```

NGLView requires a live Jupyter kernel with widget support enabled. It does not render in a static `.ipynb` viewer, on GitHub, or in nbconvert HTML output. For figures intended for the poster or a written report, write the structure to a file and render it in VESTA, OVITO or VMD, all of which produce publication-quality images that NGLView does not.

### Files written

| File | Contents |
|---|---|
| `cuo_on_hbn.xyz` / `.vasp` | Cell 1: 6×6 hBN sheet, 72 atoms, no CuO |
| `cuo_on_hbn_random_above_B_and_N_surface_fully_randomized.xyz` / `.vasp` | Cell 2: 43×43 hBN sheet, 3,698 atoms, 6 CuO |
| `cuo_inside_bnnt.xyz` / `.vasp` | Cell 3: (10,10) BNNT, 3,600 atoms, 6 CuO |

The `.vasp` files are POSCAR-format inputs for VASP. They contain unrelaxed positions, so a geometry relaxation is required before any energy from them is meaningful.

---

## 10. Verified quantities

Every number below was obtained by re-executing the notebook's geometry code, not estimated.

| Quantity | Cell 1 | Cell 2 | Cell 3 |
|---|---|---|---|
| Support | hBN sheet | hBN sheet | (10,10) BNNT |
| Supercell | 6 × 6 | 43 × 43 | 90 unit cells |
| Support atoms | 72 | 3,698 | 3,600 |
| Support mass (u) | 893.41 | 45,886.63 | 44,670.60 |
| Target CuO mass at 1 wt% (u) | 8.93 | 458.87 | 446.71 |
| CuO molecules placed | **0** | 6 | 6 |
| Box dimensions (Å) | 15.00 × 12.99 × 30.00 | 107.50 × 93.10 × 30.00 | 53.56 × 53.56 × 261.36 |
| Support area (nm²) | 1.9 | 100.1 | 94.3 (inner wall) |
| CuO areal density (nm⁻²) | 0 | 0.060 | 0.064 |
| CuO position | 2.5 Å above sheet | 12.0 Å **below** sheet | 3.0 Å from inner wall |
| Cu–O bond length (Å) | 1.73 | 1.73 | 1.73 |
| Homonuclear B–B or N–N bonds | 0% | 0% | 0% |
