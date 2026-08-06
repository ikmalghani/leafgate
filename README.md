# Leaf Gate Model

Tiny **leaf / not_leaf** gate for the ACLIS STM32 cascade (runs before the PlantVillage disease CNN).

## Contents

| Item | Description |
|------|-------------|
| [`Colab Training Run History/TRAINING_RESULTS.md`](Colab%20Training%20Run%20History/TRAINING_RESULTS.md) | Training metrics (run 2, 2026-07-28) |
| [`Colab Training Run History/run_2_2026-07-28.pdf`](Colab%20Training%20Run%20History/run_2_2026-07-28.pdf) | Colab scroll capture (current) |
| [`Colab Training Run History/run_1_2026-07-28.png`](Colab%20Training%20Run%20History/run_1_2026-07-28.png) | Earlier Colab run (superseded) |
| `aclis_leaf_gate_train_tflite.ipynb` | Train + INT8 TFLite export (Colab) — **current / baseline** |
| `aclis_leaf_gate_alt_distill_qat.ipynb` | **Alt recipe**: MobileNetV3 KD + field augs + MixUp/EMA + QAT; side-by-side vs baseline |
| `aclis_leaf_gate_96x_full_int8.tflite` | Exported INT8 model (~50.9 KB) |
| `leaf_noleaf_dataset/` + `.zip` | Kaggle leaf / non-leaf data |
| `emit_leaf_gate_c_from_tflite.py` | TFLite → namespaced C runtime |
| `codegen_leaf_gate_c.py` | Installs generated C into `ACLIS_IKMAL` |
| `verify_leaf_gate_ramp.py` | Host INT8 ramp parity vs MCU `-DLEAF_GATE_RAMP_TEST` |

## Preprocess / deploy notes

- Train: `RandomResizedCrop(96, scale=0.5–1.0)`; eval & firmware: full-frame `Resize(96)` (no center crop).
- Camera buffer is `u8-128`; firmware ImageNet-requantizes to TFLite int8 (scale≈0.01846, zp=-16) — not a raw cast.
- Parity check before blaming the model:

```bash
../tinyengine/venv/bin/python verify_leaf_gate_ramp.py
# MCU: rebuild main.cpp with -DLEAF_GATE_RAMP_TEST and match LCD scores
```

Generated / wired sources are **not** moved here so the STM32 project keeps building:

- `ACLIS_IKMAL/Src/TinyEngine/codegen_leaf_gate/`
- `ACLIS_IKMAL/Inc/leaf_gate_nn.h`
- cascade logic in `ACLIS_IKMAL/Src/main.cpp`

## Regenerate C from TFLite

```bash
cd "Ikmal/Leaf Gate Model"
../tinyengine/venv/bin/python codegen_leaf_gate_c.py
```
