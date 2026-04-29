# ZTF Embedding Demo — TODO

## Current work (in flight)

- [ ] **Raw light-curve refactor** — drop pre-binned LCs, ship raw `(bjd, flux, flux_err)` per source, add raw / phase-folded toggle in JS.
  - [x] Backup demo to `ztf-embedding-demo.fallback-2026-04-28/`
  - [x] Add `read_matchfile_lcs_batched()` to `astrotools.ztf.matchfile` (batched primitive)
  - [x] Refactor `embed_chen2020_subset.py` to use astrotools primitive
  - [x] Write `export_raw_lcs.py` + add corrupt-matchfile error containment
  - [x] Update `index.html` for raw shards + raw/folded toggle + g/r split colors
  - [ ] Re-run export (in progress, ~6 min ETA last check)
  - [ ] Smoke-test: click points, toggle raw/folded, verify shard fetch sizes

- [x] **Per-layer inference script** — `embed_chen2020_per_layer.py` written, run on the 25K subset; `per_layer.npz` is `(6, 25565, 256)`.

## Shipped (live)

- [x] **Initial 25K demo** published at <https://emmachickles.github.io/ztf-embedding-demo/>, GH Pages enabled via `gh` CLI. ~512 MB bundle.
- [x] **Quick-win UX** (TODO #1): search by RA/Dec, random source, permalink (`#idx` deep link + clipboard share), class-info modal with one-line biographies, class-aware LC marker styling (r=class color, g=lighter+triangle, i=darker+square).
- [x] **RA/Dec smooth morph** (TODO #2 Stage A): View dropdown with `Plotly.animate()` between UMAP and (RA, Dec) over 1.2 s.
- [x] **Lomb–Scargle periodogram side panel** (TODO #4 first item): JS LS on a 600-point log-spaced period grid, with catalog period + 0.5×/2× harmonic markers.
- [x] **Highlight kNN**: toggle that dims the scatter to the clicked source's 20 z_sig nearest neighbors (k-NN graph precomputed and shipped as `knn.json`, ~2.9 MB).
- [x] **Layer toggle + architecture popup** (TODO #3a): Layer dropdown shows UMAP of each transformer layer's masked-mean-pooled output (`per_layer_umap.json`, 6 layers + final z_sig); smooth animate between them. "Model" button opens an HTML/CSS architecture diagram.
- [x] **Period–luminosity diagram side panel** (TODO #4 third item): 5th right-column panel; clicking a point in P–L jumps to that source.
- [x] **Gaia DR3 RUWE + proper motions** via STILTS cross-match against `I/355/gaiadr3` for all 25K sources. Adds RUWE / pmra / pmdec / DR3 source_id to `points.json` and three new color-by axes.
- [x] **Galaxy (X–Y) bird's-eye view** (TODO #2 Stage C): galactocentric coords from `astropy.coordinates.Galactocentric` for the ~9.5K sources with parallax SNR > 5; new "Galaxy (X–Y)" view option morphs to (gx, gy) with the Sun at (−8.122, 0) marked.
- [x] **Per-class HR isolines toggle**: optional contour overlay (40/70/90% of each class's density) over the HR-diagram backdrop. Toggleable.
- [x] **Manuscript figure script** (TODO #7): `manuscript_figures.py` produces 6 publication-quality PDFs from `points.json`. Now in `ztf-ssl-transformer/figs/manuscript/`.
- [x] **Pin & Compare** for the LC panel: pin a source, then clicking another renders both LCs in the same plot (pinned dimmer/smaller). Each folds on its own period.
- [x] **Lasso → aggregate stats**: lasso-mode toggle; selecting a region opens a modal with class composition, period and r-mag stats for the selection. Works in any view.
- [x] **z_sig nearest-neighbor chips** in the info card: 6 clickable class-colored chips for direct navigation through the embedding's neighborhood (lighter alternative to the full k-NN LC strip).

## Still in the backlog

### Investigations / known issues

- **Periodogram peak doesn't always land at the catalog period.** The JS Lomb–Scargle on a 600-point log-spaced grid (0.05–50 d) has a frequency resolution near typical RRL periods of ~0.01 cycles/d, but ZTF's ~5-yr baseline can resolve down to ~5e-4. So we're undersampling frequency by ~20×. Diagnosis path:
  - Pick a specific source where the demo periodogram misses the catalog peak. Run the *same* light curve through (a) `astropy.timeseries.LombScargle` (b) astrotools' periodogram.py (c) `wd-periodicity` to check whether a peak exists in those tools at the catalog period.
  - If yes: my JS implementation has a bug, or the period grid is too coarse. Fix is either (i) refine to ~6000 periods (10× cost, still <1 s in JS), or (ii) two-pass: coarse grid first, then a fine grid in a window around each top-N peak; or (iii) seed a fine grid around the catalog period (cheap; loses generality).
  - If no: maybe pooled g+r flux is being treated wrong (e.g., median-subtracted-fractional in different units across filters). Try LS on the dominant filter only, or with proper noise weighting.
  - Reference implementations to compare:
    - `~/libraries/astrotools/astrotools/periodogram.py` (canonical).
    - `~/projects/wd-periodicity/scripts/debug/s3_diag_one_atlas_lc.py` (Lomb–Scargle helper, has been used on noisy ATLAS data).
- **Eclipse phase 0 alignment.** Currently `t_ref = min(t)` across filters → phase 0 = earliest observation, not eclipse minimum. We *do* have Chen+2020 `T0` in `chen2020.hdf5`; thread it through `points.json` and use `((t − T0) mod P) / P` so eclipses (and pulsation maxima for RR Lyrae) land at phase 0 by definition.

### Linear-probe / classification example browser

**Goal.** Tightly couple the demo to model-evaluation results. From a confusion matrix or a period-recovery scatter, click an interesting cell (false positive, false negative, or unrecovered period) and the demo opens that example so the user can interactively diagnose what went wrong.

**Data we'd need.**
- For the **classification probe** (FNN or LogReg on `z_sig` predicting var_type): per-source predicted class + softmax/probability + true class. Save a `probe_predictions.json` keyed by demo idx.
- For **period recovery** (e.g., the LS catalog vs Chen+2020 ground truth, or any model-derived period): per-source recovered period, ratio to truth, "recovered" flag. Save a `period_recovery.json` keyed by demo idx.

**UI.**
- New modal or panel: confusion-matrix heatmap. Each cell is clickable; clicking populates the lasso-style summary panel with the sources in that cell (true class i, predicted class j) and a chip list to navigate to specific examples.
- Same pattern for period recovery: scatter of `log P_recovered / P_true`. Click any source — fly to it in the embedding, show its LC + periodogram.
- A "Filter to FN/FP" toggle in the source-info card so the user can see the model's mistakes color-coded directly on the UMAP.

**Effort.** Modest — once the predictions JSON exists, the UI is two new modals + one new color-by axis. The hard part is generating the predictions cleanly: needs a stable train/test split documented alongside the export.

**Pragmatic order.** Build the predictions export first (in `ztf-ssl-transformer/scripts/`), commit them next to the embeddings, then layer the UI. Pairs especially nicely with TODO #3 (per-layer reps) — show the confusion matrix at each layer to see *where* the model starts getting class right.

### Other backlog

- **Proper-motion arrows** in RA/Dec view (sub-feature of TODO #2). Have pmra/pmdec; just needs JS quiver-style annotation overlay limited to the visible / selected subset.
- **Aladin Lite full-screen sky context** (TODO #2 Stage B). Bigger lift than Stage A — separate panel-swap, no morph.
- **Per-class Gaia DR3 vari_\*** (TODO #5). STILTS cross-match against `I/358/vrrlyrae` and `I/358/vcepheid` for class-specific physics. Would add color-by axes for [Fe/H] (RR), pulsation mode, etc. Limited to RR/RRc/CEP/CEPII subsets.
- **3D UMAP toggle** (TODO #6). Code path branches between 2D scattergl and 3D scatter3d; saved this for daylight to make sure the 2D↔3D state machine is clean.
- **k-NN LC strip** (richer version of the chips): five mini phase-folded LCs of the nearest neighbors. The chips are a starter; the strip is the upgrade.
- **Per-epoch / training-trajectory UMAPs** (TODO #3 stretch). Needs intermediate checkpoints from training.
- **Blender visualizations** (TODO #8). The reach goal.

## Backlog (reordered)

Numbering reflects current priority. Each item has my impact-vs-effort rating in parens.

### 1. Quick wins / table-stakes UX *(few hours; high impact)*

These have to exist the first time someone says "*can you show me ZTFJ052610?*"

- **Search bar** — by ZTF ID, RA/Dec, or position; flies the camera to the point.
- **Random source button** — weighted by class or z_qual; gets people exploring.
- **Permalink / share button** — URL encodes selected source + color-by + view mode.
- **Tooltip class biographies** — hover a legend entry → one-sentence description ("RR Lyrae are old, low-mass, helium-burning pulsators…").
- **Class-aware light-curve marker styling.** Currently g/r/i are encoded as fixed colors (green/red/dark-red) in the LC plot. Change to: **r = the source's class color** (matching its UMAP scatter color), **g = a slightly *lighter* shade of that class color, with a different marker shape** (e.g. open triangle), **i = a slightly *darker* shade with another shape** (e.g. open square). Add a small inline legend (`r ●  g ▲  i ■`). Makes the LC visually consistent with the scatter and conveys the filter information without wasting color budget.

### 2. Spatial views *(Stage A: half day; B & C: ~1 day each; high impact)*

Three complementary views of *where the sources are*. Each toggles via a "View" dropdown next to the existing color-by control.

- **Stage A — RA/Dec sky scatter, with smooth morph from UMAP.** `Plotly.animate()` interpolates point positions when toggled. Same scatter, just different x/y arrays. *Open question:* flat RA/Dec or Mollweide projection? Flat morphs better; Mollweide looks more sky-like.
- **Stage B — Aladin Lite full-screen sky-context.** Separate "open in sky" affordance: swap the Plotly scatter for an Aladin Lite catalog overlay (DSS / PS1 / 2MASS backdrop, native projection). No smooth morph in this mode — it's a panel swap. Reference: `~/projects/TESS-Cycle-9`.
- **Stage C — Bird's-eye Galaxy view, for parallax-constrained sources.** Convert `(ra, dec, parallax)` → galactocentric `(X, Y, Z)` (Sun at ~(−8.12 kpc, 0, 0.025), `astropy.coordinates.Galactocentric`). Show the X–Y plane: a top-down look at where these variables live in the Galaxy. Color by Z (height above plane) or by var_type. Only renders sources with parallax SNR > 5 (~37% of subset, per current data); the rest are dimmed or hidden. Astrophysically very revealing — RR Lyrae form a halo, BYDra are local-disk, EBs trace the disk.
- **Proper-motion arrows** (sub-feature, applies to A and C). For the visible subset (or just the clicked source + its k-nearest neighbors), draw small arrows showing `(pmra, pmdec)` direction & magnitude. Plotly's annotation arrows or a quiver-style line trace. Probably only practical for ≲ 1000 visible points at a time, so it auto-engages when zoomed in or for selected subsets.

**Dependencies.** Stage C and pmra/pmdec arrows need Gaia DR3 `parallax`, `parallax_error`, `pmra`, `pmdec`. We have parallax already in `chen2020.hdf5`; need to add `pmra`, `pmdec`. **Use STILTS (`~/stilts.jar`)** for the Gaia upload-cross-match — much faster than astroquery TAP for ~95K source IDs. Pattern: build a small CSV with `source_id` (or `ra`, `dec`), call `stilts cdsskymatch in=…` against `gaiadr3.gaia_source` returning the new columns.

### 3. Architecture panel + per-layer representations *(half day; highest pedagogical return)*

The "*demonstrate how the transformer learns*" feature.

- **(a) Per-layer UMAP toggle.** [Script written] Run `embed_chen2020_per_layer.py`, UMAP each layer's pooled output, save 6 sets of 2D coords. Add a Layer 0–5 + z_sig dropdown that uses `Plotly.animate()` to fly between them. Storage ~5 MB.
- **Architecture popup.** Static SVG/PNG of the model graph (input → Fourier embed → 6 transformer layers → masked-mean-pool → z_sig + z_qual heads). Probably draw.io-exported SVG.
- **Stretch — training-trajectory UMAPs.** Same toggle but at epochs 1/5/10/25/49 instead of layers. Needs intermediate checkpoints.
- **Stretch — interactive architecture.** Clicking a layer in the SVG drives the layer toggle.
- **Stretch — attention heatmaps.** Per-source attention pattern over phase bins; small fourth right-column panel.
- **Stretch — z_sig vector for clicked source as a bar / radial chart**, with class mean overlaid. Makes the abstract embedding feel like a real, finite object.

### 4. Linked side panels *(1–2 days; very high impact for monitor-scale demo)*

These turn the demo from "look at this UMAP" into "explore what the model learned." Big monitor lets us run all of them simultaneously.

- **Periodogram (LS power)** for the clicked source, with the catalog period and 0.5×/2× harmonics marked. Computed in JS from the raw LC. Pedagogically essential.
- **k-Nearest-neighbors strip.** For the clicked source, find 5–10 most similar sources by z_sig cosine distance; display their LCs in a strip. Most direct answer to "*what does the embedding think this looks like?*"
- **Period–luminosity diagram** (log P vs absolute G) with RR Lyrae and Cepheid P–L relations overlaid; clicked source highlighted.
- **Aladin Lite sky inset** at clicked RA/Dec — small always-visible panel; catches blends and crowded fields. (Different from #2's full-screen sky toggle.)
- **Lasso selection on UMAP** → aggregate sidebar: class composition, period histogram, mean phase-folded LC of the selection.
- **Pin & compare** — pin source A, click source B, render LCs in stacked panels with shared y-scale.

### 5. Per-class physical parameters from Gaia DR3 *(~1 day; high scientific payoff)*

For each variability class, surface physical-system parameters as additional color-by axes. Once the cross-match is done, every later analysis benefits.

| Class | Useful params | Source |
|---|---|---|
| RR, RRc | [Fe/H], abs G, Bailey diagram | Gaia DR3 `vari_rrlyrae` |
| DSCT | Pulsation period, amplitude, mode | Gaia DR3 `vari_short_timescale` |
| EA, EW | Mass ratio q, fillout, eccentricity, radii | Gaia DR3 `nss_two_body_orbit`, Kepler EB |
| RSCVN, BYDra | Rotation period, spot fraction | Gaia DR3 `vari_rotation_modulation` |
| CEP, CEPII | Mode, [Fe/H], abs G | Gaia DR3 `vari_cepheid` |
| Mira, SR | Mass-loss rate, AGB indicator | Gaia DR3 `vari_long_period_variable`, OGLE-IV LPV |

**Pragmatic order.** Start with `vari_rrlyrae` and `vari_cepheid` — cleanest sky-wide coverage and direct astrophysical meaning. Build out from there.

**Demo UX.** Class-aware color-by dropdown — only show params relevant to currently-filtered classes.

### 6. Visualization polish *(few hours; nice-to-haves)*

- **Similarity coloring on UMAP.** When a source is selected, optionally color all other points by cosine similarity in z_sig space.
- **Class density isolines** on the HR diagram and (later) the PL diagram. Toggleable per class — see *which class lives where* without color-confusion.
- **3D UMAP toggle.** Plotly's 3D scattergl handles 100K points fine; often shows structure 2D collapses.

### 7. Manuscript figure script *(half day; keeps demo and paper in sync)*

A standalone script that consumes the same `points.json` the demo uses and produces publication-ready static panels: HR diagram colored by class, UMAP density, P–L diagram with relations, etc. Ensures the paper and demo tell the same story even as we iterate.

### 8. (Reach) Per-source Blender-rendered model animations *(multi-day)*

Click → besides LC and HR, show a Blender-rendered animation of a physical model fit to that source's class, cycled in phase, with the model LC overlaid on the data.

| Class | Model | Implementation |
|---|---|---|
| EA, EW | Ellipsoidal binary, optionally with eclipses | `ellc`, `phoebe` |
| RR, RRc, DSCT, CEP | Radially pulsating star, R(φ) and T(φ) | Analytic Fourier of LC |
| RSCVN, BYDra | Rotating star with cool spots | Spotted-sphere model |
| Mira | AGB envelope pulsation, dust | Too heavy for live use — pre-rendered only |

**Plan.** Pre-render ~30 representative animations (3 per subtype) parameterized by 3–5 fit values from the LC. On click, the source's class picks the animation; fit values pick a variant. No live Blender call — that's the trick.

**Scale reference.** Every animation must include a familiar astronomical object for scale (Sun for stellar-radius systems, Earth for compact systems, the inner solar system for very-wide binaries). A small inset showing the Sun beside an EW contact binary tells the audience *more* about that system in one image than any number of fit values would. Pick the scale per subtype so the comparison is interpretive, not just decorative.

Reference pipeline: `~/projects/blender_astro`.

---

## My take (updated)

- **Items 1+2 are the cheapest wins.** Quick-wins UX is essentially free, and the RA/Dec morph is one `Plotly.animate()` call away. Together: ~1 day, immediately changes how the demo reads.
- **Item 3 (architecture + per-layer)** is the highest pedagogical return per hour and is what makes this a *demo of an SSL model* rather than just a UMAP browser. Per-layer UMAP toggle is the single most-bang feature to add. Inference script is already written; just needs to run.
- **Item 4 (linked side panels)** is what justifies the big-monitor format. With multiple panels visible, every click becomes a multi-axis exploration. Periodogram and k-NN strip are the highest-leverage of the four.
- **Item 5 (Gaia physical params)** is the highest scientific payoff. Slot it after the demo's interaction model is mature, so we don't keep redesigning the color-by UI.
- **Item 6** is polish — do alongside other items as time permits.
- **Item 7 (manuscript figures)** I'd touch the moment we have something publishable to show.
- **Item 8 (Blender)** is genuinely reach — defer until 1–7 are mostly in.

**If forced to pick four for a focused push:** quick-wins + RA/Dec morph + per-layer reps + periodogram. Those four reframe the demo end-to-end.
