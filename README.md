[README.md](https://github.com/user-attachments/files/28857627/README.md)
# Compositional Conformal Prediction (CoCP)

**Calibrated, transferable uncertainty for porphyry-Cu magma fertility.**

CoCP turns a fertility classifier into one that *knows when to abstain*. It computes the conformal nonconformity score, the covariate-shift weights, and the out-of-distribution test in **Aitchison geometry** (isometric log-ratio space) instead of probability or raw Euclidean space — which keeps coverage stable when the model meets a new magma series or a new district.

One self-contained notebook fetches the data, runs every experiment, and generates all figures.

## Quick start

**Colab (easiest)** — open `CuFertility_Complete_Pipeline.ipynb` -> *Runtime -> Run all*.

- `QUICK = True` (cell 0) runs a fast smoke test.
- `QUICK = False` (default) reproduces the full results (~10-15 min; faster on GPU).
- GPU section (learned-embedding CoCP): *Runtime -> Change runtime type -> GPU*.

**Local**

```bash
pip install -r requirements.txt
jupyter notebook CuFertility_Complete_Pipeline.ipynb
```

## Results at a glance

Grouped cross-validation exposes the leakage that random splits hide:

| System | random CV (AUC) | grouped CV (AUC) |
|---|---|---|
| Whole-rock | 0.909 | 0.758 |
| Zircon | 0.980 | 0.943 |

Under domain shift (transfer to a held-out magma series, coverage at a 0.90 target, 20 splits):

| Method | coverage |
|---|---|
| **CoCP (ilr)** | **0.884 +/- 0.028** |
| LAC (probability) | 0.857 +/- 0.036 |
| raw-space weighted | 0.743 +/- 0.151 |

- Minority **barren** class coverage: **0.47** (marginal) -> **0.72** (class-conditional / Mondrian).
- Unseen deposits: Quellaveco recall **0.92** (4% flagged OOD); Corcapunta recall **0.40** (**47% flagged OOD** — the model signals its own failure instead of guessing).

## Data (fetched automatically, no manual download)

- **Whole-rock** — Nathwani et al. (2022): porphyry training set, GEOROC background, and an unseen-deposit validation set. `github.com/ChetanNathwani/WRgeochem_ML`
- **Zircon** — Zou et al. (2022): fertile/barren zircons plus two unseen districts (Highland Valley, subduction; Gangdese, post-collisional). `github.com/shaohaozou/magmafertility` · Zenodo `10.5281/zenodo.6459709`

## Method in short

- **Compositional preprocessing** — major oxides as one closed subcomposition (multiplicative replacement, closure, clr/ilr); trace + REE in a separate log space.
- **Leakage-controlled evaluation** — deposit-/district-grouped cross-validation; a programmatic guard blocks target leakage in the regression track.
- **CoCP** — nonconformity = Aitchison distance (Mahalanobis in ilr) to the class centroid; coverage holds for any score under exchangeability, and the geometric score stays stable under compositional shift where raw-space importance weighting collapses.
- **OOD / abstention** — Aitchison-Mahalanobis distance thresholded at the 99% chi-squared quantile.
- **Extensions** — Mondrian (class-conditional) calibration, APS adaptive sets, an OOD-gated hybrid Guard, a deep-ensemble Bayesian baseline, and a GPU learned-embedding variant.

## Repository contents

```
CuFertility_Complete_Pipeline.ipynb   # the full pipeline — run this
cufertility_complete_pipeline.py      # same pipeline as a script
requirements.txt
README.md
```

Running the notebook writes `figures/` (figures 1-4) and `results/` (coverage tables, per-deposit validation, headline summary), then bundles everything into a timestamped `.zip`.

## Citation

If you use this code, please cite the data sources (Nathwani et al. 2022; Zou et al. 2022) and this repository.

## License

Released under the MIT License — see `LICENSE`.
