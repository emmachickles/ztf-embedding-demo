# ZTF Light Curve Embedding Explorer

Interactive in-browser demo of self-supervised transformer embeddings learned
from Zwicky Transient Facility (ZTF) light curves. Each point in the scatter
plot is a periodic variable star; click one to see its raw or phase-folded
light curve and its position on the Gaia HR diagram. Recolor the embedding by
class, period, magnitude, or learned noise score to probe how the learned
representation organizes physical structure versus survey systematics.

> **Status:** prototype slice (~25 K labeled periodic variables, 11 types)
> released alongside an in-progress research project on representation
> learning for time-domain surveys. The full-scale results are not yet public.

## Live demo

**<https://emmachickles.github.io/ztf-embedding-demo/>**

## What you are looking at

- **Embedding.** A 2D UMAP projection of `z_sig`, the signal-channel
  representation produced by a self-supervised transformer trained on
  multi-band ZTF light curves.
- **Sample.** 25,565 sources drawn from the [Chen et al. 2020 ZTF Catalog of
  Periodic Variable Stars](https://ui.adsabs.harvard.edu/abs/2020ApJS..249...18C/abstract),
  stratified across 11 variability types (RR Lyrae sub-types, eclipsing
  binaries, δ Scuti, RS CVn, BY Dra, semi-regulars, Cepheids, Miras…). Rare
  classes are kept in full; common ones are capped at 3,000 each.
- **Light curves.** ZTF *g* and *r* photometry from the public matchfile
  archive, with standard quality cuts. The viewer can show the raw multi-band
  series or fold it on the Chen+2020 catalog period.
- **HR diagram.** Backdrop is a log-density grid of the *full* Chen+2020
  catalog (~636 K sources with parallax SNR > 5); the clicked source is
  highlighted as a colored marker.
- **Color-by options.** Variability type, period, *r* magnitude, `z_qual`
  (a learned per-source noise score that often tracks survey systematics
  rather than astrophysics), Gaia BP–RP color, and absolute Gaia G.

## Run locally

The demo is a static site. From the repo root:

```bash
python3 -m http.server 8000
```

Then open <http://localhost:8000>.

## Files

| File | Size | Description |
|------|------|-------------|
| `index.html` | ~16 KB | Plotly-based UI; lazy-loads light-curve shards on click |
| `points.json` | ~6 MB | Per-source records: UMAP coords, var_type, period, rmag, z_qual, Gaia mags / parallax / abs G, PS1 g/r/i, RA/Dec |
| `manifest.json` | small | Shard count, t_ref, UMAP parameters |
| `hr_background.json` | ~190 KB | 2D log-density grid of the full Chen+2020 catalog on the HR diagram |
| `lightcurves_raw/shard_NNNN.json` | ~5–10 MB each | Per-source raw `(bjd, flux_normalized, flux_err)` for both *g* and *r* bands; ~50 shards of 500 sources |

## Data provenance & attribution

- Periodic-variable labels and ephemerides: Chen et al. 2020, *ApJS* 249, 18.
- Photometry: ZTF Data Release products. Use of ZTF data should follow the
  [ZTF data acknowledgement](https://www.ztf.caltech.edu/page/data-policy).
- Astrometry & broadband mags: Gaia DR3 and Pan-STARRS DR2.
- Embeddings, `z_qual`, and the UMAP projection: produced by this project.

## Citing this demo

If you reference the demo in talks, papers, or applications, please cite the
repository. A `CITATION.cff` file will be added once the associated
manuscript is on arXiv.

```
@software{ztf_embedding_demo,
  author  = {Chickles, Emma},
  title   = {ZTF Light Curve Embedding Explorer},
  year    = {2026},
  url     = {https://github.com/emmachickles/ztf-embedding-demo}
}
```

## License

Code is released under the [MIT License](LICENSE). The bundled data products
are derivatives of public ZTF observations and the Chen et al. 2020 catalog;
please cite those upstream sources when reusing the data.
