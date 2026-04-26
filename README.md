# ZTF Light Curve Embedding Explorer

Interactive in-browser demo of self-supervised transformer embeddings learned
from Zwicky Transient Facility (ZTF) light curves. Each point in the scatter
plot is a periodic variable star; click one to see its phase-folded light
curve. Recolor the embedding by class, subtype, period, magnitude, or noise
quality to probe how the learned representation organizes physical structure
versus survey systematics.

> **Status:** prototype slice (~14k labeled periodic variables) released
> alongside an in-progress research project on representation learning for
> time-domain surveys. The full-scale results are not yet public.

## Live demo

**<https://emmachickles.github.io/ztf-embedding-demo/>** *(enable GitHub
Pages on the `main` branch to publish)*

## What you are looking at

- **Embedding.** A 2D UMAP projection of `z_sig`, the signal-channel
  representation produced by a self-supervised transformer trained on phase-
  folded ZTF light curves.
- **Sample.** 14,341 sources drawn from the [Chen et al. 2020 ZTF Catalog of
  Periodic Variable Stars](https://ui.adsabs.harvard.edu/abs/2020ApJS..249...18C/abstract),
  approximately balanced across three classes:
  - RR Lyrae (RRL): 4,783
  - Eclipsing Binaries (EB): 4,863
  - δ Scuti (DSCT): 4,695
- **Light curves.** Each source's ZTF *r*-band photometry was phase-folded on
  the catalog period and binned into 50 phase bins.
- **Color-by options.** Class label, subtype (RR / RRc / EA / EW / DSCT),
  period, *r* magnitude, and `z_qual` — a learned per-source noise score that
  often tracks survey or systematic effects rather than astrophysics.

## Run locally

The demo is a single static page that loads two JSON files. Any static file
server works. From the repo root:

```bash
python3 -m http.server 8000
```

Then open <http://localhost:8000>.

## Files

| File | Size | Description |
|------|------|-------------|
| `index.html` | 10 KB | Plotly-based UI; loads both JSON files at startup |
| `points.json` | 1.6 MB | Per-source records: `x`, `y` (UMAP coords), `label`, `var_type`, `period`, `rmag`, `z_qual`, `idx` |
| `lightcurves.json` | 33 MB | Per-source phase-folded curves keyed by `idx`: `phase`, `flux`, `err` (each length 50), and `n_obs` |

## Data provenance & attribution

- Periodic-variable labels and ephemerides: Chen et al. 2020, *ApJS* 249, 18.
- Photometry: ZTF Data Release products. Use of ZTF data should follow the
  [ZTF data acknowledgement](https://www.ztf.caltech.edu/page/data-policy).
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
