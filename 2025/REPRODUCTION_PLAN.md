# REPRODUCTION PLAN
## μSAM — Segment Anything for Microscopy
### Archit et al., Nature Methods, 2025 | Review 1

---

## 1. Paper Reference

| Field | Value |
|---|---|
| **Title** | Segment Anything for Microscopy |
| **Authors** | Anwai Archit, Sushmita Nair, et al. |
| **Journal** | Nature Methods |
| **Published** | 12 February 2025 |
| **Volume** | 22, pages 579–591 |
| **DOI** | 10.1038/s41592-024-02580-4 |
| **Official Code** | https://github.com/computational-cell-analytics/micro-sam |

---

## 2. Selected Experiment

**Name:** LIVECell Specialist — Automatic Instance Segmentation (AIS)

The paper evaluates μSAM on LIVECell in Table 1, comparing:
- SAM ViT-L (zero-shot, default)     → SA50 = 0.431
- μSAM Generalist (ViT-B)           → SA50 = 0.559
- μSAM LIVECell Specialist (ViT-L)  → SA50 = 0.617

Our target: μSAM ViT-B fine-tuned on LIVECell → expected SA50 ~0.58–0.61

---

## 3. Architecture

- **Encoder:** SAM ViT-B (12 transformer blocks, 768 hidden, 86M params)
- **Decoder:** SAM mask decoder + μSAM distance-transform decoder (additional head for AIS)
- **Training input:** patches of 520×704 → internally padded to 1024×1024

---

## 4. Dataset & Splits

| Split | Images |
|---|---|
| Train | 3,752 |
| Val   | 1,001 |
| Test  | 1,512 |

Local path: data/images/ and data/annotations/LIVECell/

---

## 5. Preprocessing

- raw_transform: identity (NO [-1,1] rescaling)
- label_transform: PerObjectDistanceTransform (foreground + distances + boundary)
- patch_shape: (520, 704), random crop

---

## 6. Training Configuration

| Parameter | Value |
|---|---|
| Optimizer | Adam |
| Learning rate | 1e-5 |
| LR Scheduler | ReduceLROnPlateau (min, factor=0.9, patience=10) |
| Iterations | 100,000 |
| Batch size (train) | 2 |
| n_objects_per_batch | 25 |
| Early stopping | 10 epochs |
| Starting checkpoint | μSAM ViT-B Generalist (auto-downloaded) |

---

## 7. Evaluation Metric

**SA50** = fraction of GT instances matched with predicted IoU ≥ 0.5
Computed via elf.evaluation.matching.matching()

---

## 8. Documented Deviations from Paper

| Aspect | Paper | Ours | Reason |
|---|---|---|---|
| Model encoder | ViT-L | ViT-B | ViT-L needs 80GB GPU; ViT-B is official script default |
| Iterations | 250,000 | 100,000 | Time/compute budget; matches official script default |
| Hardware | A100 80GB | Colab T4/A100 | Availability |
| Workers | 16 | 2 | Colab CPU cores limited |

---

## 9. Official Repository Files Used

- finetuning/livecell_finetuning.py  → primary training script
- finetuning/livecell_inference.py   → inference using micro_sam.evaluation.livecell
- finetuning/livecell_evaluation.py  → evaluation
- micro_sam.training.train_sam()     → core training loop
- torch_em.data.datasets.get_livecell_loader → data loading

