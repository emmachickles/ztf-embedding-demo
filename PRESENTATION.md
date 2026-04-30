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
  baseline. On the 11-class subset (5-fold CV, balanced accuracy):

  | Features | Probe | Bal. acc. |
  |---|---|---|
  | z_sig only | linear | 48% |
  | z_sig + log(P) + rmag | linear | 67% |
  | 9 classical features (period, mags, amps, R21, ϕ21, Ng, Nr) | RF | **87%** |
  | z_sig + classical | RF | 76% |

  Adding z_sig to the classical features does *not* help RF. The
  current SSL model is essentially re-deriving information already in
  Chen+2020's summary statistics. That's a failure of training setup,
  not of the SSL approach in principle: we phase-fold inputs on the
  catalog period, so the model sees one cycle and captures *shape* but
  not period or aperiodic features.

  The case for continuing on SSL despite this is precisely *what
  classical features can't do*: discover novelty, encode aperiodic
  signatures, handle blends, and transfer across surveys. The next
  experiment removes the phase-fold crutch — the model trains on raw,
  unfolded sequences and has to recover period structure on its own.
  That's where SSL stops being redundant with classical features.

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

## How to practice — a 2-week run-up plan

The tighter you can make the demo, the better. The mistake is to read
the script once and call it done; the goal is *muscle memory* on the
30-second pitch and *confident improvisation* on the 5- and 10-min.

### Day 1 — read & map

1. Read `PRESENTATION.md` end-to-end, slowly. Don't try to memorize.
2. Open the live demo on a regular screen and click through each
   feature *while reading the matching section of the script*. Notice
   any beat where the script doesn't match what's on screen — those
   are the places that'll trip you up live.
3. Make a list of every claim with a number in it (e.g., "25 K
   sources", "48% balanced accuracy", "11 classes", "100 K training
   set"). Verify each against `TODO.md` so you're not surprised.

### Days 2–3 — lightning pitch, internalize

The 30-sec pitch should be muscle memory by the end of day 3.

1. Practice it out loud, looking at a mirror or your laptop camera, 5
   times in a row. The lines you stumble over are signals to rewrite.
2. Then do it 5 more times *without the screen* — stand up, walk
   around. The pitch should survive without the visual.
3. Time yourself. 30 seconds is tight. If you're at 45+ s, trim
   words, not concepts.
4. Finally, practice the **one-sentence version** ("self-supervised
   transformer on ZTF, learns variable-star structure with no labels,
   en route to LSST"). You'll need this for hallway intros.

### Days 4–6 — 5-min walkthrough, with feedback

1. Run through the 5-min script once *with* the live demo, watching
   the clock. First pass will be ~7 minutes. Don't worry, that's
   normal.
2. Identify *which beats* ran long. Often it's the periodogram or the
   architecture popup — they're pretty, but spending 2 min there
   eats your budget.
3. Cut anything that doesn't directly serve "this model has learned
   astrophysics from raw light curves with no labels." Anything else
   is bonus material for the 10-min version.
4. Get an audience. **One person you trust** is enough. Ideally one
   physicist *and* one non-physicist; their questions are different
   and you need both rehearsals.
5. Ask them to interrupt with questions. Note which questions threw
   you. Pre-script answers for the top 3.

### Days 7–10 — 10-min deep dive, anticipate Q&A

1. Run the 10-min version through twice, once for an astronomer
   audience and once for a tech/ML audience. Adjust emphasis: for
   astronomers, dwell on the HR/PL diagrams and the period-recovery
   issues. For ML folks, dwell on the architecture, loss, and the
   shape-without-period diagnosis.
2. Read the Q&A bank in `PRESENTATION.md` and **say each answer out
   loud**. Some sound great in writing and clunky out loud — fix
   those.
3. Add 2–3 questions you anticipate from your specific audience,
   write your answers in the bank, rehearse them.

### Days 11–13 — dress rehearsal on the actual hardware

This is the most-skipped step and the one that pays the most.

1. Connect to the Vibe Board exactly as you will on the day. Note
   the actual viewport size, latency to GitHub Pages, and any
   keyboard / clicker oddities.
2. Run the full 10-min on the Board. Things that look fine on a
   laptop can be illegible at 8 ft viewing distance — you may want
   to tweak fonts or zoom.
3. Try the demo with bad-network simulation (Chrome DevTools → Slow
   3G) to know what happens if the venue WiFi is flaky. Have the
   local fallback ready.
4. Test all keyboard shortcuts on your actual presentation laptop's
   keyboard. The `?` key in particular requires Shift on US English.

### Day 14 — fresh and minimal

1. Don't over-rehearse the morning of. Run the 30-sec pitch twice,
   skim the Q&A bank once, walk away.
2. Do open the live URL once to confirm it's serving. If GitHub
   Pages is down, switch to the local `python3 -m http.server` copy
   ahead of time.
3. Bookmark the PR/issue/manuscript URL you'll point people to for
   follow-up. Be ready to give *one* concrete next step for anyone
   who's interested.

### Crutches that help

- **A rolling timer.** Keep a phone with a stopwatch visible during
  practice. After day 5 you should know "this is the 90-second mark"
  by the section you're on.
- **A clicker (or `↓` on the keyboard).** Scrolling with a clicker
  feels natural; clicking individual points needs a real mouse.
- **A throat lozenge.** Demos rely on your voice; presentations cause
  dry mouth.
- **A backup window with the demo already loaded.** If your live
  browser dies, you have a second one ready.
- **A canned source.** Pre-load the URL with `#7` (an RR Lyrae) so
  the demo opens with a sensible initial selection if you arrive at
  the podium under time pressure.

### What to avoid

- Reading from the script during the talk. You'll lose eye contact
  and sound robotic. The script is a scaffold — practice until you
  can ad-lib around it.
- Promising the SSL representation is "better than classical." It
  isn't, yet (probe results are honest in the Q&A). Frame it as
  "different and more general"; you'll lose credibility if you
  oversell.
- Live-debugging on stage. If a feature looks broken, name it briefly
  ("…that side panel doesn't always render the contours, ignore
  it…") and move on. Audiences forgive issues; they don't forgive
  derailments.
- The first 5 seconds of silence while you find your spot. Open with
  the lightning pitch *while* the demo loads — start strong.

## Backup talking points (if something breaks)

- Live demo at the local URL `http://localhost:8765/` is identical to the
  GH Pages version.
- The fallback dir at `~/projects/ztf-embedding-demo.fallback-2026-04-28/`
  is the pre-overhaul demo if you need to revert during the talk.
- Raw light curves can be re-fetched on the fly via the IRSA ZTF API if
  the bundled shards fail.

