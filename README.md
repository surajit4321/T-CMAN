# T-CMAN: audio-visual deepfake detection with cross-modal attention

Code, per-seed results and figure scripts that accompany the manuscript
*"T-CMAN: Temporal Cross-Modal Attention Network for Audio-Visual Deepfake Detection"*
(S. Paul, A. Chakraborty, B.P. Devi; submitted to *Computer Optics*).

The repository lets you re-run the experiments of Section 6 of the paper: a model with frozen EfficientNet-B4 and
Wav2Vec 2.0 backbones, a bidirectional-GRU temporal encoder, an 8-head cross-modal attention bridge and an auxiliary
audio-visual synchrony-contrastive objective, trained on **LAV-DF** and tested within-dataset and, with
**FakeAVCeleb** held out entirely, across datasets.

## What to expect (headline results)

| | AUC (mean ± SD, 5 seeds) |
|---|---|
| Within-dataset (LAV-DF test, group-disjoint split) | 0.9990 ± 0.0003 |
| Cross-dataset (LAV-DF → FakeAVCeleb), full model | 0.650 ± 0.063 |

Cross-dataset AUC of six configurations (five seeds each):

| Configuration | Cross-dataset AUC | Δ vs. full (pts) | Welch p | Holm p |
|---|---|---|---|---|
| full | 0.650 ± 0.063 | – | – | – |
| no_sync (λ_sync = 0) | 0.693 ± 0.039 | +4.29 | 0.236 | 1.00 |
| no_adv (λ_adv = 0) | 0.664 ± 0.051 | +1.32 | 0.724 | 1.00 |
| no_crossmodal (concatenation) | 0.668 ± 0.021 | +1.77 | 0.576 | 1.00 |
| no_temporal (no GRU) | 0.682 ± 0.065 | +3.19 | 0.452 | 1.00 |
| sync_strong (λ_sync = 1.0) | 0.640 ± 0.044 | −1.08 | 0.761 | 1.00 |

**These are negative / inconclusive results and should be read that way.** Within-dataset detection is near-perfect,
but cross-dataset transfer is weak and varies by up to 13 AUC points between seeds of the *same* configuration.
No ablation differs significantly from the full model (one-way ANOVA p = 0.56), and the proposed synchrony objective
shows no detectable benefit. With five seeds, only differences of about 9 AUC points or more can be detected, so
"no effect detected" does not mean "no effect". Robustness to compression and to adversarial attacks (Gap 2 of the paper) is **not** evaluated.

## Repository layout

```
notebooks/   01 dataset audit -> 02 feature extraction -> 03 model and training -> 04 cross-dataset sweep and statistics
scripts/     stats_tests.py (reproduces the paper's tests from results/), make_figures.py (all figures)
results/     per-seed results (seed_results.json), training curve, partitions, test table (CSV)
configs/     hyper-parameters (config.yaml) and the six configurations (ablations.yaml)
figures/     the figures of the paper, vector PDF and 300-dpi PNG
splits/      clip ids per partition (written by notebook 03; commit after running it)
docs/        DATA.md (obtaining and laying out the data), ARCHITECTURE.md, RELEASE_CHECKLIST.md
```

## Quick check without any data

```bash
pip install -r requirements.txt
python scripts/stats_tests.py     # reproduces the ablation statistics (Tables 6-7) from results/seed_results.json
python scripts/make_figures.py    # regenerates Figures 1-5 into figures/
```

## Full reproduction

1. **Environment.** Python 3.10, CUDA-enabled PyTorch, `pip install -r requirements.txt`, and system FFmpeg (`sudo apt install ffmpeg`).
2. **Data.** Obtain LAV-DF and FakeAVCeleb from their official sources and place them under `data/raw/` as described in [docs/DATA.md](docs/DATA.md). The datasets are **not** redistributed here.
3. **Notebook 01**, run twice (`DATASET = "lavdf"`, then `"fakeavceleb"`): audits each dataset and writes `data/feats/<dataset>/manifest.json`. The audit stops with an explanation if labels or group ids cannot be parsed.
4. **Notebook 02**, run twice with the same settings: caches frozen EfficientNet-B4 and Wav2Vec 2.0 features (hours on a single GPU for LAV-DF).
5. **Notebook 03**: builds the group-disjoint split (with leak assertions), trains the baseline and writes `splits/lavdf_splits.json`.
6. **Notebook 04**: loads notebook 03 via `%run`, trains six configurations × five seeds (about 50–80 min per run on an RTX A5000; one to two days in total; resumable), evaluates within- and cross-dataset, and prints the statistics.

Save the notebooks under exactly these file names; notebook 04 runs `03_model_and_training.ipynb`.

## Reproducibility notes

* Every training run takes an explicit `seed` that controls weight initialisation, data order, augmentation and perturbation noise (`train_model(..., seed=s)`, `make_loader(..., seed=s)`). In our check, repeating a seed reproduced the within-dataset test metrics exactly. Cross-dataset AUC varies much more between seeds than within-dataset metrics do, which is the central observation of the paper.
* Results were produced on one NVIDIA RTX A5000 (24 GB) with mixed precision. Exact numbers can differ slightly on other hardware because of non-deterministic GPU kernels.
* Backbones are frozen and their outputs are cached once, so all configurations see identical inputs.
* `results/seed_results.json` holds the numbers reported in the paper. Within-dataset AUC for the ablations is stored for seeds 0–2 only; notebook 04 as shipped stores within-dataset metrics for every seed.
* The notebooks use the class name `SyncAVNet` for the model that the paper calls T-CMAN (see [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)).

## Known limitations (details in Section 8 of the paper)

LAV-DF labels are collapsed to whole-clip real/fake and its group-disjointness is a surrogate (no speaker ids); LAV-DF has one generation pipeline, so no held-out-method test is possible; our copy of FakeAVCeleb has 475 real and 21,085 fake clips; backbones are frozen; only one dataset pair and direction (LAV-DF → FakeAVCeleb) is evaluated; FaceForensics++ could not be used because the copy we obtained had no audio streams.

## Citation and licence

Please cite the paper (see `CITATION.cff`; to be updated after publication). Code is released under the MIT licence (`LICENSE`).
LAV-DF and FakeAVCeleb are third-party datasets under their own terms; please obtain them from the original authors.
