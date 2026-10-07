# Paper modules and code

The paper's model (T-CMAN) is implemented in `notebooks/03_model_and_training.ipynb` as `SyncAVNet`
(the name used during development).

| Paper | Code | Notes |
|---|---|---|
| M1 visual encoder | `timm` EfficientNet-B4, notebook 02 | frozen, cached, 1792-d per frame |
| M2 audio encoder | torchaudio `WAV2VEC2_BASE`, notebook 02 | frozen, cached, 768-d, pooled to 30 windows |
| M3 temporal encoder | `TemporalEncoder` | linear → 2-layer Bi-GRU (256/direction) → LayerNorm → linear; `use_temporal=False` keeps only the input projection |
| M4 cross-modal bridge | `CrossModalBridge` | two 8-head attention modules (V→A, A→V), concatenation + projection; `use_crossmodal=False` replaces it by concatenation |
| M-Sync | `sync_contrastive_loss` | InfoNCE on real clips, temperature 0.07, hard negative = audio shifted by 2–8 windows; training only |
| M5 classifier | `SyncAVNet` head | mean-pool over time, MLP 512→256→2 |
| Adversarial consistency | `compute_losses` | one modality chosen per batch with equal probability, Gaussian noise σ = 0.05 on its cached features, MSE between softmax outputs |
| Training | `train_model(cfg, ..., seed=)` | AdamW, one-cycle schedule, 40 epochs, best validation AUC checkpoint |
| Metrics | `evaluate` | AUC, F1, recall, precision, accuracy, equal error rate |
