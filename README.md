# fda-geostat-eeg

Functional geostatistical analysis of resting-state EEG spectra in normal cognition, mild cognitive impairment (MCI) and dementia.

This repository contains the analysis code for:

> Ospina-Galindez, J. A., López, J. A., Jaramillo-Jimenez, A., Ochoa-Muñoz, A. F., Nuñez, J. M., & Giraldo Henao, R. *Functional geostatistical data analysis of resting-state EEG spectral activity in mild and major neurocognitive disorder* (manuscript in preparation).

## Pipeline

`fda_geostat_eeg.ipynb` runs the whole analysis:

1. **Geometry**: 10–20 template positions, great-circle inter-electrode distances, hemisphere-symmetric spherical Delaunay adjacency.
2. **Preprocessing** (per subject, cached): eyes-closed segments from the event files (≥ 60 s), 1–45 Hz band-pass, average reference, autoreject, ICA (Picard) + ICLabel, local autoreject, 15 epochs of 4 s selected with scorEpochs.
3. **Spectra**: multitaper PSD (1–30 Hz, bandwidth 1 Hz), median across epochs, log10; penalized B-splines with GCV.
4. **Age**: functional residualization of the spline coefficients (non-linear, diagnosis-specific).
5. **Geostatistics**: trace-variograms under four functional dissimilarities (L2, Wasserstein, Mahalanobis, Fisher–Rao), five variogram models, practical range h95 with identifiability checks, observed-domain endpoints.
6. **Inference**: Kruskal–Wallis / Mann–Whitney with effect sizes; MNE cluster-based permutation tests (cluster mass and TFCE) at sensor level and in frequency × electrode space.
7. **Sensitivity analyses** and export of tables, figures and `results_macros.tex`.

## Data

The CAUEEG dataset is **not included** and is not publicly downloadable. Access must be requested from its owners; see the [CAUEEG repository](https://github.com/ipis-mjkim/caueeg-dataset) and cite:

> Kim, M.-J., Youn, Y. C., & Paik, J. (2023). Deep learning-based EEG analysis to classify normal, mild cognitive impairment, and dementia: Algorithms and dataset. *NeuroImage*, 272, 120054. https://doi.org/10.1016/j.neuroimage.2023.120054

Expected layout:

```
caueeg-dataset/
├── signal/edf/{serial}.edf
├── event/{serial}.json
└── dementia-no-overlap.json
```

## Running

**Google Colab** (CPU runtime; "High-RAM" recommended for more cores): open the notebook, set `BASE_DIR` in Section 1 to the dataset folder in your Drive, and run all cells.

**Locally**:

```bash
pip install -r requirements.txt
export CAUEEG_BASE_DIR=/path/to/caueeg-dataset
export FDA_OUT_DIR=/path/to/results        # optional
jupyter nbconvert --to notebook --execute fda_geostat_eeg.ipynb --output executed.ipynb
```

Optional environment variables: `FDA_N_JOBS` (parallel workers), `FDA_N_PERM` (default 5000), `FDA_N_PERM_3D` (default 1000), `FDA_N_BOOT` (default 500) and `FDA_N_SUBJECTS_MAX` (quick tests).

Preprocessing is the slowest step (roughly 1–2 h on 8 cores) and is cached per subject (`cache_v23_subjects/`), so an interrupted run resumes where it stopped.

## Outputs

Written to `results_v23/` inside the dataset folder (or `FDA_OUT_DIR`):

- `figures/`: main and supplementary figures (PDF and PNG);
- `tables/`: all tables (CSV and LaTeX);
- `results_macros.tex`: every number reported in the manuscript, as LaTeX macros.

## License

MIT (see `LICENSE`).
