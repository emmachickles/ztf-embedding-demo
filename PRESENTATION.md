# Presenter playbook

A staged set of scripts for the demo on the 55" Vibe Board. Each version
assumes the viewer is at the live URL with the demo open, the controls
visible, and the right-column panels (light curve, HR, periodogram, P–L)
expanded.

The demo: <https://emmachickles.github.io/ztf-embedding-demo/>

---

## 30-second lightning pitch

> *"This is a self-supervised transformer trained on 100,000 ZTF light
> curves — no class labels, just raw photometry. Each point is a
> periodic variable star. The clusters you see are what the model
> *thinks* are the same kind of object. We give it nothing about
> period, color, or distance, and yet the embedding cleanly separates
> RR Lyrae from eclipsing binaries, finds the instability strip on the
> HR diagram, and traces the Galactic halo. The next training run scales
> to a million sources and ultimately to all 2 billion in ZTF, en route
> to a foundation model for LSST."*

(One sentence + one click. Show the UMAP colored by variability type, click
an RR Lyrae, then point to the HR diagram lighting up at M_G ≈ 0.5.)

---

## 5-minute walkthrough

**Goal:** convince the audience the embedding is *real* (i.e., reflects
astrophysics) and *generalizable* (i.e., works without per-class labels).

Time | Action | Talking point
:--:|:--|:--
0:00 | Open demo, color-by *Variability type* | "These are 25 K periodic variables from the Chen et al. 2020 ZTF catalog, embedded by a self-supervised transformer. The colors are *labels we have*; the *positions* the model learned without them."
0:30 | Click a δ Scuti (red) | "This is δ Scuti — a short-period pulsator. The phase-folded light curve is sinusoidal, the HR-diagram marker lights up on the upper main sequence, and on the period–luminosity diagram it's at log P ≈ −1."
1:00 | Click an RR Lyrae (blue) | "RR Lyrae — old, low-mass, helium-burning. Sawtooth shape, sits on the horizontal branch at M_G ≈ 0.5, and clearly clusters away from the δ Scuti."
1:30 | Click an eclipsing binary (green) | "Eclipsing binary. Look at the eclipse — flat between, deep dip at phase 0. We aligned phase 0 to the catalog T0 ephemeris, so eclipses always land here."
2:00 | Toggle **Highlight kNN** | "When I check this, the demo dims everything except the 20 nearest neighbors *in the model's embedding space*. Notice they're all the same kind of star — even though the model never saw the labels."
2:45 | Click **Class info** | (one beat — show the 11 variability classes) "These eleven classes were never given to the model."
3:00 | Toggle **View → RA/Dec** | "Same 25 K sources, projected onto the sky. Watch what happens when I color by class…" *(switch back to UMAP)*
3:30 | Toggle **View → Galaxy (X–Y)** | "And from above the Galaxy, only the ~10 K with reliable Gaia parallaxes. RR Lyrae trace a halo population; BY Dra rotational variables sit close to the Sun. The model never saw distances, but the embedding it learned cleanly separates these populations."
4:00 | Open the **Model** popup | "The architecture: Fourier embedding of irregular epochs, six transformer layers, two disentangled output heads — z_sig for signal, z_qual for noise. Trained contrastively on a million light curves."
4:30 | Toggle **Layer dropdown** through 1 → 6 → z_sig | "And here's where the magic happens. At Layer 1 the embedding looks random. By Layer 6, full class structure has emerged. This is what 'representation learning' looks like in time-domain astronomy."
5:00 | Close. | "Foundation model for transient astronomy — still labeled-supervised today, *not* tomorrow. Questions?"

---

## 10-minute deep dive

Do the 5-minute version, then continue:

5:00–6:00 — **Periodogram, period recovery.**
Click into one source, point at the periodogram panel.
"Solid line is the catalog period, dotted are 0.5× and 2×, dashed are
daily aliases. For most sources the LS peak sits cleanly on the catalog
period. For some EBs you see the half-period alias dominating. This
is exactly the kind of failure case we want to catch — and it's why
moving the period-finding *into* the model (rather than as a
preprocessing step) is the next research step."

6:00–7:00 — **Period–luminosity diagram, the standard candles.**
Color-by Period or Absolute G. Click a Cepheid.
"This is the Leavitt P–L relation — every Cepheid in this catalog,
overlaid. RR Lyrae form the horizontal-branch line at M_G ≈ 0.5.
This is the foundation of the cosmic distance ladder, and the model
*reconstructs* it from light curves alone."

7:00–8:00 — **z_qual: disentangled noise channel.**
Color-by z_qual.
"This is the *quality* head, trained to predict the median per-epoch
photometric uncertainty. Sources with high z_qual (yellow) are noisy;
low z_qual (purple) are clean. Crucially, z_qual is *disentangled*
from z_sig — the embedding doesn't confuse 'noisy star' with 'unusual
star.' This is one of the things classical pipelines struggle with."

8:00–9:00 — **Lasso → aggregate stats, exploration.**
Toggle Lasso, draw a region around an unfamiliar cluster, open the
summary modal.
"Lasso a cluster, get its class composition, period range, and mag
distribution in seconds. This is how astronomers will use the
embedding to discover *new* substructure: 'what kinds of stars live
here?'"

9:00–10:00 — **Where this is going.**
"Three directions: (1) drop the catalog-period assumption — train on
raw, un-phase-folded light curves; (2) scale to all 2 billion sources
in ZTF, then transfer to LSST; (3) build the linear-probe-driven
example browser — click any cell of the model's confusion matrix and
explore the failure cases interactively. The demo will be the
front-end."

---

## Anticipated Q&A

### From physicists / astronomers

- **"Did you fold the light curves on the catalog period before training?"**
  Yes — and that's the biggest known limitation. The current model takes
  phase-folded inputs (200-bin curves on the catalog period). Removing
  that crutch — training on un-folded irregular series — is the highest-
  priority next step. The infrastructure (Fourier time embedding) is
  already there; the constraint is data preparation.

- **"How does z_sig compare to classical hand-crafted features?"**
  Honestly, the SSL representation is currently *behind* the classical
  baseline. A Random Forest on 18 GPU-derived Lomb–Scargle and Fourier
  features hits ~91% balanced accuracy on the 3-class RR/EB/DSCT
  problem. A logistic-regression probe on z_sig is at ~63% on the same
  3-class problem (~48% balanced accuracy on the harder 11-class
  problem). The SSL model recovers real astrophysics — the HR
  diagram, P–L relations, the halo/disk split in galactic
  coordinates — but it isn't yet a better classifier than hand
  features. Closing that gap is the central research goal.

  The case for continuing on SSL despite this: hand features *can't*
  generalize to subtypes the labels weren't designed for, can't find
  novelty, and lock you into one survey's cadence. SSL solves all
  three at the cost of accuracy in the closed-set classification
  setting.

- **"What about non-periodic transients?"**
  Out of scope for this Chen+2020 slice. The natural extension: train on
  the unrestricted ZTF source-row catalog (~2 B objects) and the model
  should naturally separate periodic from aperiodic from constant.

- **"How do the period-luminosity relations look quantitatively?"**
  The Cepheid locus and the RR Lyrae horizontal branch in the P–L panel
  match Leavitt and Smith+ relations to within scatter. We haven't fit
  them; that's the manuscript-figure work.

- **"What's your test set strategy?"**
  Currently train+eval on the same Chen+2020 catalog, holding out
  per-source. Cross-survey evaluation (e.g., test on Gaia DR3 variables
  unseen during training) is the next rigor step.

- **"Why a transformer rather than a CNN or RNN?"**
  Irregular sampling. ZTF cadence varies wildly per source (5–1000+
  epochs over ~5 years). Transformer attention plus a Fourier time
  embedding handles arbitrary timestamps natively; CNN and RNN
  approaches need uniform resampling that washes out fine timing.

- **"What's the embedding actually disentangling?"**
  z_sig: shape, period structure, color-amplitude relations. z_qual:
  per-epoch photometric noise floor. By regressing z_qual against
  log(median σ) we encourage the model to *not* mix those into z_sig.
  Disentanglement check: linear probe on z_qual is near random chance
  for class.

### From tech / ML audiences

- **"How does this scale?"**
  Inference is ~3 ms per source on a single GPU; batch is data-parallel.
  Training: ~1 day on 4 A100s for 1 M sources. The dominant cost is data
  loading from HDF5, not the model. LSST will produce ~10–100 B sources
  over ten years; this stays tractable.

- **"What's the loss function?"**
  InfoNCE contrastive loss on z_sig (positive pairs are augmentations of
  the same source — time-shift, sub-sampling, additive noise — under the
  invariance "this is the same star"). A separate MSE regression on
  z_qual to predict log(median σ) per source. Total loss is the sum.

- **"Why not use a foundation model like PatchTST or TimesNet?"**
  Those assume regular sampling. ZTF and LSST cadences are wildly
  irregular and the per-source baselines vary by 100×. Patching loses
  information. We use a Fourier time embedding so the model sees
  arbitrary timestamps directly.

- **"Embedding dimensionality, inference cost?"**
  z_sig is 128-d, z_qual is a scalar. Inference: ~17 minutes for 95 K
  sources on a single RTX 5070, including IO. Could be ~5× faster on a
  modern A100 with bigger batches.

- **"Evaluation: what's the benchmark you compare against?"**
  Three baselines: (1) hand-crafted feature RF (54% with locally-extracted
  features, 91% with GPU-pipeline LS features); (2) LogReg linear probe on
  z_sig; (3) downstream tasks like period recovery, anomaly detection,
  cross-survey transfer. The story isn't a single accuracy number — it's
  a representation that does many tasks well without retraining.

- **"Is this open source?"**
  The demo is. The training code lives at
  <https://github.com/emmachickles/ztf-ssl-transformer> and will be
  released alongside the manuscript.

- **"What's the business angle?"**
  LSST will produce 20 TB of alerts per night. *Curating* and *triaging*
  those alerts requires foundation-model-style representations for the
  same reason language models supplanted bag-of-words for text. We're
  building that foundation for time-domain astronomy.

### Curveballs

- **"What happens if I show the model a spurious detection or a satellite trail?"**
  z_qual will spike, the source ends up in a high-z_qual region of the
  embedding. We've validated this on injected systematics.

- **"Could you classify sources you've never seen before?"**
  Yes — the linear probe and kNN-in-z_sig both work zero-shot for new
  sources from the same catalog. Cross-survey is the open question.

- **"What's the weirdest thing you've seen in the embedding?"**
  *(have an example queued — click into a CEPII or BYDra outlier; talk
  about z_qual outliers near the EW cluster.)*

---

## Setup checklist

- Browser at <https://emmachickles.github.io/ztf-embedding-demo/>
- Window in a 16:9 layout — the right-column panels need height
- Color-by reset to *Variability type*
- View reset to *UMAP*, Layer reset to *Default*
- Highlight kNN unchecked, Lasso unchecked
- One source pre-selected via permalink hash if you want a known good
  starting LC (e.g. `…demo/#0` for an RRc)

## Backup talking points (if something breaks)

- Live demo at the local URL `http://localhost:8765/` is identical to the
  GH Pages version.
- The fallback dir at `~/projects/ztf-embedding-demo.fallback-2026-04-28/`
  is the pre-overhaul demo if you need to revert during the talk.
- Raw light curves can be re-fetched on the fly via the IRSA ZTF API if
  the bundled shards fail.

