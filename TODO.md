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

- [x] **Per-layer inference script** — `embed_chen2020_per_layer.py` written (uses forward hooks on each transformer encoder layer + masked-mean-pool). Not yet run; will produce `(n_layers, n_sources, d_model)` tensor for TODO #3.

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
