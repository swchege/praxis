# VAE Boundary Synthesis for Zero-Day Intrusion Detection

Code for the doctoral praxis *Adversarial Generalization in Deep Learning–Based Network Intrusion Detection Systems* (Steven Chege, The George Washington University).

The praxis asks whether synthetic records generated near class boundaries in the latent space of a Variational Autoencoder (VAE) help a deep learning intrusion detection system recognize attacks it was never trained on. One attack class is withheld from each of three benchmark datasets as a zero-day proxy. A VAE-augmented CNN-LSTM is then compared with an identical CNN-LSTM trained without synthetic data and with a Random Forest.

The pipeline runs as five Jupyter notebooks, designed for Google Colab or Jupyter Notebook with a GPU.

---

## Hypotheses

| | Hypothesis | Test |
|---|---|---|
| **H1** | VAE augmentation reduces the Generalization Gap, G = F1_known − F1_zero-day | Paired bootstrap, 98.3% CI on ΔG = G_B − G_A, with a 0.02 known-class F1 margin |
| **H2** | The augmented model has higher zero-day recall than both baselines | Recall at a 1% benign false-alarm rate; exact McNemar test (α = 0.05/3); Cohen's h |
| **H3** | The classifier explains real and synthetic records of each attack class the same way | Spearman ρ between SHAP feature rankings > 0.80 for every class, with real-vs-real (ceiling) and real-vs-benign (floor) controls |

## Results in brief

| | CIC-IDS2017 | UNSW-NB15 | NSL-KDD |
|---|---|---|---|
| H1 | Not supported | Not supported | Not supported |
| H2 | Not supported | Not supported | Not supported |
| H3 | Not met | Met | Met |

With the full 20% synthetic budget met on every dataset, augmentation did not improve zero-day detection. The Random Forest was the strongest zero-day detector on all three datasets. Full results and discussion are in Chapter 4 of the praxis.

---

## Repository contents

```
notebooks/
├── Praxis Step 1-2 - Preprocessing (Praxis held-out classes).ipynb
├── Praxis Step 3 - VAE Training (Praxis held-out classes).ipynb
├── Praxis Step 4 - Synthetic Generation (Praxis held-out classes).ipynb
├── Praxis Step 5-6 - Classifiers (Praxis held-out classes).ipynb
└── Praxis Step 7-9 - Evaluation (Praxis held-out classes).ipynb
README.md
```

| Notebook | Pipeline steps | What it does |
|---|---|---|
| Step 1-2 | Preprocessing | Loads the raw data, removes duplicates, holds out the zero-day class, splits into train / validation / standard test / held-out sets, and fits all encoding, feature filtering and scaling on the training split only |
| Step 3 | VAE training | Sweeps the latent dimension (d_z ∈ {8, 16, 32, 64}) and trains the final VAE on known-class training data, with KL annealing (β: 0 → 1 over 50 epochs) |
| Step 4 | Synthetic generation | Generates candidates by tail, interpolation and noise sampling, filters them through a three-stage quality gate (projection, reconstruction, SHAP), and tops up until 20% of the training set is synthetic |
| Step 5-6 | Classifier training | Runs the CNN-LSTM grid search, then trains the Random Forest, Experiment A (real data) and Experiment B (real + synthetic) with seeds 42, 43 and 44 |
| Step 7-9 | Evaluation | Tests H1, H2 and H3 and writes all results and an HTML report |

### Held-out (zero-day) classes

| Dataset | Held-out class | Held-out records (after de-duplication) |
|---|---|---|
| CIC-IDS2017 | Web Attack – SQL Injection | 21 |
| UNSW-NB15 | Shellcode | 1,446 |
| NSL-KDD | buffer_overflow, loadmodule, perl, rootkit (U2R) | 88 |

---

## Getting started

### 1. Download the datasets

The datasets are not included in this repository. Download them from their publishers and place each in its own folder under `datasets/` in your project folder:

| Dataset | Source | Files | Folder |
|---|---|---|---|
| CIC-IDS2017 | [Canadian Institute for Cybersecurity](https://www.unb.ca/cic/datasets/ids-2017.html) | `MachineLearningCSV.zip` (the eight daily CSV files) | `datasets/CIC-IDS2017/` |
| UNSW-NB15 | [UNSW Sydney](https://research.unsw.edu.au/projects/unsw-nb15-dataset) | `UNSW_NB15_training-set.csv`, `UNSW_NB15_testing-set.csv` | `datasets/UNSW-NB15/` |
| NSL-KDD | [Canadian Institute for Cybersecurity](https://www.unb.ca/cic/datasets/nsl.html) | `KDDTrain+.txt`, `KDDTest+.txt` | `datasets/NSL-KDD/` |

### 2. Set up the project folder

The notebooks expect this layout in Google Drive:

```
MyDrive/ColabNotebooks/GWUPraxis/
├── datasets/
│   ├── CIC-IDS2017/
│   ├── UNSW-NB15/
│   └── NSL-KDD/
└── (notebooks)
```

To use a different location, set the `PRAXIS_BASE` environment variable or edit `BASE` near the top of each notebook.

### 3. Run the notebooks in order

1. Step 1-2 – Preprocessing
2. Step 3 – VAE Training
3. Step 4 – Synthetic Generation
4. Step 5-6 – Classifiers
5. Step 7-9 – Evaluation

Run each notebook from top to bottom. The first code cell mounts Google Drive and installs dependencies. Each notebook reads the outputs of the previous one from `outputs_praxis_heldout/`. Steps 3 and 5-6 resume after a Colab disconnect: run the notebook again from the top.

---

## Outputs

All outputs are written to `GWUPraxis/outputs_praxis_heldout/`:

| Folder | Contents |
|---|---|
| `pipeline/` | Data splits, de-duplication log, class counts, class-distribution figures |
| `VAETrain/` | VAE weights, latent-dimension sweep, training curves, active-unit diagnostic |
| `SynthGen/` | Accepted synthetic records, quality-gate report, `retrain_flags.json` |
| `classifier/` | Trained models, training log, grid-search results |
| `evaluation/<run id>/` | Hypothesis results and the full report |

Key files in `evaluation/<run id>/`:

| File | Contents |
|---|---|
| `evaluation_report.html` | All results on one page |
| `per_run_metrics.csv` | Every metric for every model and seed |
| `seed_summary_mean_sd.csv` | Mean and SD of every metric over the seeds |
| `h1_results.json`, `h1_by_seed.csv` | H1 for the validation-selected runs and for each seed |
| `h2_results.json`, `paired_tests_by_seed.csv` | H2 recall, McNemar tests and Cohen's h |
| `h3_results.csv` | Per-class SHAP correlations with ceiling and floor controls |
| `heldout_outcomes_by_seed.csv` | Outcome 1 (top prediction not benign) and Outcome 2 (Synthetic-OOD) |
| `selected_runs.json` | The seed selected on validation macro-F1 for each model |
| `*_beeswarm.png`, `*_profiles.png` | SHAP plots for each evaluated class (augmented CNN-LSTM) |

---

## VAE retraining rule

The protocol calls for the VAE to be retrained when round-one quality-gate acceptance falls below 85%. This step is manual. Step 4 writes the names of datasets below the threshold to `retrain_flags.json`. To apply the rule:

1. Copy those names into `RETRAIN_RULE_DATASETS` in Step 3.
2. Rerun Step 3, then Step 4 and the later notebooks.
3. Report both acceptance rates.

In the run reported in the praxis, all three datasets were flagged, but the rule was not applied. The praxis reports this as a protocol deviation (Section 4.6.5).

---

## Environment

Runs reported in the praxis used Google Colab with a single NVIDIA GPU.

| Library | Version |
|---|---|
| Python | 3.11 |
| TensorFlow / Keras | 2.21.0 |
| scikit-learn | 1.6.1 |
| pandas | 2.2.3 |
| NumPy | 2.1.3 |
| SHAP | 0.54.1 |
| SciPy | 1.16.3 |
| Matplotlib | 3.10.0 |
| joblib | 1.6.0 |

**Run time.** CIC-IDS2017 is the largest workload: one CNN-LSTM training epoch takes about 105 seconds, so a 100-epoch run takes about three hours. Steps 4 and 5-6 are the slowest stages.

**Reproducibility.** Seeds 42, 43 and 44 are fixed for weight initialization, batch shuffling and dropout. The reported run for each model is chosen on validation macro-F1, never on test results. Colab updates its runtime over time, so library versions may differ from those above; the versions in use are printed at the start of each run.

---

## Citation

If you use this code, please cite the praxis:

```bibtex
@phdthesis{chege2026praxis,
  author = {Chege, Steven},
  title  = {Adversarial Generalization in Deep Learning-Based Network Intrusion Detection Systems},
  school = {The George Washington University},
  type   = {Doctoral praxis},
  year   = {2026}
}
```

Please also cite the datasets: Sharafaldin et al. (2018) for CIC-IDS2017, Moustafa and Slay (2015) for UNSW-NB15, and Tavallaee et al. (2009) for NSL-KDD.

## License

<!-- Choose a license, e.g. MIT, and add a LICENSE file. -->
Add a license before making the repository public.
