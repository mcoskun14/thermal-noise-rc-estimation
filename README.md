# Thermal-noise RC parameter estimation

Code and data for the manuscript

> **Physics-Informed Hybrid Neural Estimation of Resistor-Capacitor Circuit Parameters from Thermal Noise Power Spectral Density**
> Musab Coşkun, Kamuran Dilsiz (manuscript under review)

The equivalent resistance *R* and capacitance *C* of a one-pole RC circuit are estimated from the power spectral
density (PSD) of its own Johnson-Nyquist thermal noise, without external excitation. The proposed estimator has two
stages:

1. a multilayer perceptron (MLP) predicts log *R*<sub>eff</sub> and log *C*<sub>eff</sub> from the log-PSD and four
   physics-guided features;
2. a per-sample maximum a posteriori (MAP) inversion refines this prediction with a calibrated, bin-averaged
   Johnson-Nyquist forward model, with the network output as the prior and the temperature as a latent variable
   when the sensor is noisy or absent.

## Repository layout

```
thermal-noise-rc-estimation/
├── data/
│   ├── RC_dataset_python.mat     simulated PSD dataset (33 MB, see data/README.md)
│   └── README.md                 variables, grid and file format
├── notebooks/
│   └── rc_noise_hybrid_estimation.ipynb   full study: data, models, inversion, CRLB, tables, figures
├── requirements.txt
├── CITATION.cff
├── LICENSE                       MIT (code)
└── LICENSE-DATA                  CC BY 4.0 (data)
```

## Compared estimators

| ID | Method |
|----|--------|
| M0 | scikit-learn MLP (original baseline) |
| M1 | matched PyTorch MLP (ReLU) on the physics features |
| M2 | twin network (two independent branches with LayerNorm, GELU, dropout), data loss only |
| M3 | M2 + spectral physics loss during training (physics-informed neural network ablation) |
| M4 | nonlinear least-squares fit of the calibrated forward model (classical system identification) |
| **M5** | **hybrid: M1 prior + per-sample MAP refinement with the forward model (proposed)** |
| M5-M2 | the same refinement on the M2 backbone |
| M2-raw | M2 on the raw log-PSD only (feature ablation) |

All neural models share the inputs, data splits, optimiser, schedule, early stopping and training budget.

## Experimental design

* **Splits:** `random`; `heldout` (four interior *R* values and *C* ∈ {0.22 µF, 220 µF} never seen in training);
  `ood_T` (train *T* < 350 K, test *T* ≥ 350 K).
* **Temperature information:** exact; Gaussian sensor error σ<sub>T</sub> = 5, 10, 20 K; none.
* **Repetitions:** 10 seeds for random split / exact temperature, 6 seeds for every other configuration; mean ± standard
  deviation and paired two-sided Wilcoxon signed-rank tests.
* **Metrics:** MAPE, R², log-R² and mean absolute log10 error; Cramér-Rao lower bounds for identifiability.

## How to run

**Locally**

```bash
pip install -r requirements.txt
cd notebooks
jupyter notebook rc_noise_hybrid_estimation.ipynb
```

The notebook looks for `RC_dataset_python.mat` in `../data/` (or `data/`). A CUDA GPU is recommended; the full study
(15 split/scenario configurations, all methods) takes several hours on one GPU and is resumable: results are appended
to `v3_results.csv` per (split, scenario, seed) block and finished blocks are skipped on re-run.
Set `RUN_STUDY = False` in the configuration cell to only rebuild the tables and figures from an existing results file.

**On Kaggle**

Upload `data/RC_dataset_python.mat` as a Kaggle dataset, attach it with *Add Input*, enable a GPU accelerator and run
all cells; the file is found automatically under `/kaggle/input`.

## Outputs

* `v3_results.csv` - one row per (split, scenario, seed, method, parameter) with MAPE, R², log-R², log-MAE
* `preds/` - per-sample predictions used for the scatter and generalisation figures
* tables (main results, paired tests, identifiability regimes, temperature scenarios, generalisation, cost) and figures
  printed / saved by the analysis cells

## Citation

If you use the code or data, please cite the article (details will be added after publication) or use
`CITATION.cff`.

## License

Code: MIT License (`LICENSE`). Data: Creative Commons Attribution 4.0 International (`LICENSE-DATA`).
