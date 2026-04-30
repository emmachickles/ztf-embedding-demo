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
4:00 | Point at the **architecture diagram** on the left. | "The architecture: Fourier embedding of irregular epochs, six transformer encoder layers, two disentangled output heads — z_sig for signal, z_qual for noise. Trained contrastively on 100,000 light curves with no class labels."
4:30 | Click through the **Layer boxes** in the architecture diagram (1 → 6 → z_sig). Watch the **accuracy meter** below. | "The accuracy meter on the left shows the linear-probe balanced accuracy at each layer. Layer 1 already has 48% — class structure is mostly there from the start. Each transformer layer adds ~0.5 to 1 point: Layer 6 peaks at 51.3%. **But watch what happens when I click z_sig** — the final head, the thing the model was actually trained to optimize, drops accuracy back to 47.9%. The encoder *learned* the structure; the contrastive projection *threw some of it away* because it was optimizing for similarity, not classification. That's a real, measurable failure mode of contrastive heads — and it's the kind of finding the demo lets you see in seconds rather than weeks."
5:00 | Close. | "Foundation model for time-domain astronomy — still labeled-supervised today, *not* tomorrow. Questions?"

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

6:30–7:00 — **The encoder learns; the head loses information.**
Walk over to the architecture mini-diagram on the left. Click each Layer
box, calling out the meter value as it climbs.
"This is the linear probe at each encoder layer. Layer 1 starts at 48%,
each subsequent layer adds half a point or so, Layer 6 peaks at 51.3%.
Now click the z_sig box, the final 128-d projection. The accuracy
*drops* back to 47.9%. The model was trained on contrastive loss, not
classification — so the head is solving a different optimization
problem than 'separate the classes,' and it loses about 3 percentage
points of class structure when it does. The pretty UMAP clusters
visualize z_sig because that's what the contrastive head exposes for
similarity, but if your downstream task is classification, the *encoder
output* is what you want, not z_sig. This is a lesson that scales to
every contrastive foundation model — see the literature on linear-probe
mismatch in CLIP or DINO."

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

- **"Why does the linear-probe accuracy *drop* from Layer 6 to z_sig?"**
  Because the head wasn't optimized for classification. The encoder's
  Layer 6 output is 256-d and class-separable to 51.3% balanced
  accuracy. The z_sig MLP head projects that to 128-d under InfoNCE
  contrastive loss, which optimizes for *cosine similarity to
  augmented views*, not for class boundary separation. Some
  class-discriminative information lives along directions that
  contrastive doesn't preserve. Same pattern shows up in CLIP and
  DINO — the encoder representation generally outperforms the
  projection-head output on linear probes. Practical takeaway: for
  downstream classification, freeze the encoder and probe Layer 6,
  not z_sig. The demo's left-panel meter shows this in real time as
  you click through the Layer dropdown.

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

## What the audience sees on the Vibe Board

When you open the demo on the 55″ screen (viewport ≥ 1400 px wide), the
layout is **three columns**:

- **Left column** — model performance dashboard (always visible):
  - Architecture mini-diagram (clickable layer boxes that drive the
    Layer dropdown)
  - **Probe accuracy of selected layer** — big-format LogReg balanced
    accuracy meter that updates live as you click through layers
  - Linear-probe confusion matrix (clickable cells)
  - Class distribution + per-class F1
- **Center** — the UMAP scatter, controls, search bar, view/layer
  dropdowns
- **Right column** — per-source detail (lights up on click):
  - Source-info card with kNN chips
  - Light curve (raw / phase-folded toggle, pin & compare)
  - Gaia HR diagram (with optional per-class isolines)
  - Lomb–Scargle periodogram (catalog period + 0.5×/2× / daily-alias
    markers)
  - Period–luminosity diagram
  - k-NN strip — 5 mini phase-folded LCs of the nearest neighbors

On a laptop (< 1400 px) the left column collapses; the same content is
still reachable via the modal buttons (Confusion / Model).

## Setup checklist

- Browser at <https://emmachickles.github.io/ztf-embedding-demo/>
- Window in a 16:9 layout — the right-column panels need height
- Color-by reset to *Variability type*
- View reset to *UMAP*, Layer reset to *Default*
- Highlight kNN unchecked, Lasso unchecked
- One source pre-selected via permalink hash if you want a known good
  starting LC (e.g. `…demo/#0` for an RRc)

## How to practice

A concrete checklist. Each step has a clear action and a clear "done"
criterion. Run them in order; skip any you've already nailed. Don't try
to do all of them in one session — most need ~30 minutes.

### 1. Read the playbook end-to-end (~30 min)

Open `PRESENTATION.md` and read it cover to cover, including the Q&A
bank. Don't try to memorize anything yet — you're just building a
mental map of what's there.
**Done when:** you can answer "where is the 5-min script?", "where's
the answer to the 'how does this compare to classical features?'
question?", and "what's in the left column of the Vibe Board layout?"
without scrolling.

### 2. Verify every number in the script against `TODO.md` (~15 min)

Open `TODO.md` next to `PRESENTATION.md` and find the source of each
number you'll say out loud:
- 25,565 sources / 11 classes (TODO header / class distribution)
- 48% balanced probe accuracy (Probe ablation table)
- 51.3% peak at Layer 6 (left-panel meter / probe table)
- 91% classical RF baseline (probe upper-bound table)
- 100 K training set, 4 hours on 1 GPU, 17 min inference (lightning
  pitch + Q&A "How does this scale?")
**Done when:** every numeric claim in the playbook traces to a number
in `TODO.md` or the live demo. If any are wrong, fix the playbook now.

### 3. Walk the live demo while reading the 5-min script (~30 min)

Open <https://emmachickles.github.io/ztf-embedding-demo/> at full screen
and click through each beat of the 5-min script *while* reading it.
Note every beat where the script doesn't match what you see — those
are exactly the moments that will trip you up live.
**Done when:** every beat in the 5-min script has a clean
"click here, say this" mapping, and you've fixed any drift.

### 4. Lock the lightning pitch — first 10 reps (~30 min)

Stand up. Read the 30-second pitch out loud, looking at your laptop
camera (not the screen). Repeat 5 times. Then close the laptop and do
5 more *while walking around the room* — the pitch needs to survive
without a visual aid.
**Done when:** you've delivered the pitch 10 times without checking
the script and without a major stumble. The lines you keep tripping
on are signals to rewrite — change them in `PRESENTATION.md`, then
re-do those reps.

### 5. Time the lightning pitch (~10 min)

Phone stopwatch. Three timed runs. Target ≤ 30 s.
**Done when:** all three runs land between 25 and 32 seconds. If you
keep going long, cut filler words ("essentially", "actually", "kind
of"). If you keep going short, you're rushing — slow the cadence on
"no labels, just raw photometry" — that's the line that needs to land.

Also memorize the **one-sentence version** for hallway intros:
*"Self-supervised transformer on ZTF, learns variable-star structure
with no labels, en route to LSST."*

### 6. First 5-min run-through — for *flow*, not timing (~25 min)

Run the full 5-min script with the live demo. Don't watch the clock.
Just get the click-and-talk pattern smooth.
**Done when:** you can do the whole walkthrough without backtracking
or saying "wait, hold on." Expect this first pass to take 6–8 min.

### 7. Second 5-min run-through — *timed*, identify long beats (~25 min)

Phone stopwatch. Run the full script again. Note which beats ran
long. Almost certainly: the periodogram (4:00 in the 10-min) and the
architecture diagram are pretty and easy to over-narrate. The Layer
toggle / probe-meter beat is the *climax* — protect its time.
**Done when:** you've cut at least one minute of script and can land
the 5-min run-through within 30 s of 5:00.

### 8. Run the 5-min for one trusted listener and capture their hard
   questions (~45 min)

Find one person — a labmate, a partner, a friend who'll tell you the
truth. Do the 5-min live, *out loud*, with their full attention. Tell
them up front: "interrupt me with questions whenever something
isn't clear."
**Done when:** you've completed the run and *written down* the three
questions that threw you. Add them to the Q&A bank if they aren't
already there.

### 9. Pre-script the top 3 hard questions into the Q&A bank (~30 min)

For each of the three questions from step 8: write a 2–3 sentence
answer into the Q&A section of `PRESENTATION.md`. Read your written
answer out loud. If it sounds clunky, rewrite. The most common
clunk: too much hedging ("well, it kind of depends...") — instead
state your answer first, then qualify.
**Done when:** all three answers can be delivered out loud in under
30 s each without reading them.

### 10. 10-min walkthrough for an astronomer-shaped audience (~45 min)

Run the 10-min script with emphasis on the astrophysics:
- Spend extra time on the HR diagram (instability strip, horizontal
  branch).
- Walk through the Period–Luminosity panel slowly — point at the
  Cepheid locus and the RRL horizontal branch.
- Discuss the period-recovery issues (catalog vs LS peak, EB
  half-period aliases).
- Less time on the contrastive loss math.
**Done when:** you can do the run without notes and the timing lands
within 30 s of 10:00.

### 11. 10-min walkthrough for an ML/tech audience (~45 min)

Same script, different emphasis:
- Open the architecture popup early. Talk through the disentangled
  heads.
- Spend time on the **Layer dropdown + accuracy meter** — that's the
  ML-audience climax. Click each Layer in the architecture mini, watch
  the meter rise to 51.3% at Layer 6 and *drop* to 47.9% at z_sig.
  Connect this to "linear-probe mismatch with the projection head"
  in CLIP/DINO.
- Discuss the InfoNCE loss, augmentations, batch size.
- Less time on individual sources.
**Done when:** you can do the run without notes; the head-vs-encoder
moment lands cleanly.

### 12. Read the Q&A bank out loud, every entry (~30 min)

Some answers read well in writing and clunk out loud. Read each Q&A
entry aloud once. The ones that feel awkward — rewrite them.
**Done when:** every Q&A answer can be delivered without you wincing
at the phrasing.

### 13. Dress rehearsal on the actual Vibe Board (~60 min)

This is the most-skipped step and the one that pays the most. Connect
to the Vibe Board exactly as you will on the day.
- Confirm the three-column layout actually appears (left panels show
  at viewport ≥ 1400 px).
- Step back 8 feet from the screen and look at the demo. If anything
  is illegible at that distance, zoom in (Cmd/Ctrl-+) until it isn't.
  Note the zoom level you settle on.
- Run the full 10-min on the Board. Time it; it'll feel different
  than on a laptop.
- Tap each keyboard shortcut on the *actual presentation laptop's
  keyboard* — `?` needs Shift on US English; some clickers steal the
  arrow keys.
**Done when:** the demo looks legible from the back of the room, and
you've done one full 10-min run on the Board without surprises.

### 14. Failure-mode rehearsal (~20 min)

Open Chrome DevTools → Network → throttling → Slow 3G. Reload the
demo. Click around. Things that take 200 ms on fast WiFi take 5–10 s
on Slow 3G — note the panels that visibly stutter or take long to
load (LC shards are the slowest, ~10 MB each).

Have a fallback ready:
- Local copy at `http://localhost:8765/` (run `python3 -m http.server
  8765` from the demo directory).
- The pre-overhaul fallback dir at
  `~/projects/ztf-embedding-demo.fallback-2026-04-28/` if even local
  is broken.

**Done when:** you know which features degrade gracefully and which
don't, and you have a rehearsed line for each ("the LC will load in
a moment — let me click into the HR diagram while we wait").

### 15. Day-of warmup (≤ 15 min, *not* more)

The morning of the talk, do *less* not more.
1. Run the 30-second pitch twice. No more.
2. Skim the Q&A bank once. Don't drill.
3. Open the live URL to confirm it's serving (look for a 200 in
   browser DevTools).
4. Bookmark or copy *one* concrete follow-up link (the
   ztf-ssl-transformer GitHub URL or your manuscript draft) so you
   can hand it to anyone who asks "where can I read more?"
5. Walk away from the laptop. Drink water. Don't rehearse again.

**Done when:** the demo is up, you have one follow-up link in hand,
and you're not visibly rehearsing.

---

### Crutches that help (use during practice and the talk)

- **A rolling timer.** Phone with a stopwatch visible during practice.
  By the time you've done step 7 twice you should be able to feel
  "this is the 90-second mark" by which beat you're on.
- **A clicker (or `↓` on the keyboard).** Scrolling needs a clicker;
  clicking specific points needs a real mouse — bring one.
- **A throat lozenge.** Demos rely on your voice and venue lights
  cause dry mouth.
- **A backup browser window with the demo already loaded.** If your
  primary tab dies you can pivot in seconds.
- **A canned source.** Open the demo with `…demo/#7` so it lands on
  an RR Lyrae and the right column is already populated when you
  start speaking.

### What to avoid

- **Reading from the script during the talk.** You'll lose eye
  contact and sound robotic. The script is a scaffold — practice
  until you can ad-lib around it.
- **Promising the SSL representation is "better than classical."** It
  isn't, yet — the probe results in the Q&A bank are honest about
  this. Frame it as "different and more general"; you lose
  credibility if you oversell.
- **Live-debugging on stage.** If a feature looks broken, name it
  briefly ("…that contour overlay doesn't always render, ignore it…")
  and move on. Audiences forgive issues; they don't forgive derailments.
- **5 seconds of silence at the start.** Open with the lightning
  pitch *while* the demo loads — start strong, don't wait for
  Plotly to finish drawing.

## Backup talking points (if something breaks)

- Live demo at the local URL `http://localhost:8765/` is identical to the
  GH Pages version.
- The fallback dir at `~/projects/ztf-embedding-demo.fallback-2026-04-28/`
  is the pre-overhaul demo if you need to revert during the talk.
- Raw light curves can be re-fetched on the fly via the IRSA ZTF API if
  the bundled shards fail.

