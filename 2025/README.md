# μSAM — Review 1: LIVECell Specialist Reproduction

**Reference:** Archit et al., "Segment Anything for Microscopy", Nature Methods 2025  
**DOI:** https://doi.org/10.1038/s41592-024-02580-4  
**Official Code:** https://github.com/computational-cell-analytics/micro-sam

---

## Project Structure

```
2025/
├── README.md                          ← This file
├── REPRODUCTION_PLAN.md               ← Full technical plan
├── REVIEW1_RESULTS.md                 ← Generated after experiment runs
│
├── data/                              ← LIVECell dataset (3.4 GB)
│   ├── images/
│   │   ├── livecell_train_val_images/ ← 4,753 TIFF images (train+val)
│   │   └── livecell_test_images/      ← 1,512 TIFF images (test)
│   └── annotations/LIVECell/
│       ├── livecell_coco_train.json   ← 551 MB
│       ├── livecell_coco_val.json     ← 97 MB
│       └── livecell_coco_test.json    ← 261 MB
│
├── configs/
│   └── livecell_vit_b.yaml            ← Training config (mirrors official script)
│
├── notebooks/
│   └── review1_musam_livecell.ipynb   ← MAIN ALL-IN-ONE NOTEBOOK (run this!)
│
└── results/
    ├── checkpoints/                   ← Saved model weights
    ├── predictions/                   ← Per-image instance masks (after Stage 5)
    ├── metrics/                       ← SA50 CSV (after Stage 6)
    └── figures/                       ← All visualization plots
```

---

## Quick Start (Colab)

1. Upload the `2025/` folder to Google Drive
2. Open `notebooks/review1_musam_livecell.ipynb` in Colab
3. **Runtime → Change runtime type → GPU (T4 or better)**
4. Run all cells from top to bottom
5. Results saved in `results/`

---

## Experiment Summary

We reproduce the **LIVECell Specialist (Automatic Instance Segmentation)** experiment from Table 1 of the μSAM paper.

| Method | SA50 |
|--------|------|
| SAM ViT-L (zero-shot) | 0.431 |
| μSAM Generalist ViT-B | 0.559 |
| **μSAM LIVECell Specialist ViT-L (PAPER)** | **0.617** |
| **μSAM LIVECell Specialist ViT-B (OURS)** | *Run experiment* |

### Key Deviations from Paper
| Aspect | Paper | Ours | Reason |
|--------|-------|------|--------|
| Encoder | ViT-L | **ViT-B** | Memory constraint |
| Iterations | 250,000 | **100,000** | Time budget |
| Hardware | A100 80GB | **Colab GPU** | Availability |

---

## Dependencies

```bash
pip install micro-sam torch-em elf tifffile matplotlib pandas seaborn
```

The `micro-sam` package automatically downloads the pretrained generalist checkpoint (~400 MB).

---

## Metric

**SA50** (Segmentation Accuracy at IoU ≥ 0.5):  
Fraction of ground-truth cell instances correctly matched by a predicted mask with IoU ≥ 0.5.  
Computed using `elf.evaluation.matching.matching()`.
