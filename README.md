# Universal_VLA


# Universal-VLA: Training-Free UI Grounding via Closed-Form Similarity Scoring

## ACL  Rebuttal 

This repository contains **complete, reproducible code** for all experiments reported in our rebuttal. All results can be verified by running the notebooks on a single A100 GPU.

---

## Repository Structure

```
├── README.md                              ← You are here
├── Universal_VLA_Rebuttal_part-1.ipynb    ← Main evaluation (start here)
├── Universal_VLA_VS_OSATLAS.ipynb         ← Complementarity experiment

```

---

## File Guide for Reviewers

### 📓 `Universal_VLA_Rebuttal_part-1.ipynb` — Main Evaluation

**Start here.** This notebook runs the complete Universal-VLA pipeline on ScreenSpot-v2 (1,272 samples) and ScreenSpot-Pro (1,581 samples).

| Cell | What it does | Corresponds to |
|------|-------------|----------------|
| 1-4 | Setup, dependencies, download ScreenSpot-v2 | 
| 5 | **Load dataset with FIXED bbox format** (`[x,y,w,h]` → `[x1,y1,x2,y2]`) Bug fix in rebuttal |
| 6 | Load models: CLIP ViT-B/32, SBERT, EasyOCR (**170M params, zero UI training**) | 
| 8 | Multi-strategy proposal engine (contour + edge + MSER + OCR) | 
| 9 | Embedding functions — implements **Eq. 3** (vision) and **Eq. 4** (text) | 
| 10 | Z-score calibration — implements **Eq. 5** | 
| 11 | Universal-VLA inference — implements **Eq. 6** (max-fusion gate) |
| 13 | **Full evaluation results**: Mobile 45.5%, Desktop 49.7%, Web 43.2%, Overall **45.8%** | 
| 15 | **Ablation: Fusion strategies** — vision-only, text-only, mean, max |  
| 16 | **Ablation: k-NN sensitivity** (k=3,5,7) — stable at 45.8-46.1%  |
| 18 | **Ablation: α sensitivity** (0.0, 0.15, 0.30) — stable at 45.7-45.9% | 
| 19 | **Score distributions**: σ_v=0.019, σ_t=0.163 (validates Z-score normalization) | 
| 20 | **Cold-start ablation**: 45.8% with empty EVM | 
| 21-27 | **ScreenSpot-Pro evaluation**: 11.6% (GPT-4o gets 0.9%) | 

**Key result:** 45.8% point-in-box accuracy with ~170M frozen parameters, zero UI training, no cloud API.

---

### 📓 `Universal_VLA_VS_OSATLAS.ipynb` — Complementarity Experiment

This notebook proves that Universal-VLA (Eq. 3-6) captures **fundamentally different grounding signals** than OS-Atlas-7B (7B parameters, finetuned on 13M UI elements).

**What it contains:**

**1. Threshold Search** — Tests whether Eq. 3-6 can be used as a confidence-based verifier to override OS-Atlas predictions. At threshold τ=10, the hybrid achieves 84.0% (+0.2% over Atlas).

**2. Complementarity Matrix** — The core result:

| Category | Count | % |
|----------|------:|---:|
| Both correct | 538 | 42.3% |
| OS-Atlas only | 529 | 41.6% |
| **Universal-VLA only** | **44** | **3.5%** |
| Both wrong | 161 | 12.7% |
| **Oracle (either correct)** | **1,111** | **87.3%** |

→ Universal-VLA uniquely solves **44 samples** that OS-Atlas-7B fails on
→ Oracle reaches **87.3%** (+3.4% over Atlas alone)
→ ManiCoG (ICLR 2026) uses GPT-5 for only +1.0%

**3. Text/Icon Breakdown** — Shows WHY the two methods are complementary:
- Universal-VLA: strong on text (66-72%), weak on icons (12-18%)
- OS-Atlas: strong on both, but UVLA rescues ~4% on text elements
- Different failure modes = complementary paradigms

---

## Key Claims Verified by This Code

| Claim | Evidence | File |
|-------|----------|------|
| 45.8% with 170M params, zero UI training | Cell 13 output | Part-1 |
| 98.5% proposal recall | Cell 13 output | Part-1 |
| Max-fusion exceeds both branches (49.7% > 47.6%, 35.0%) | Cell 15 output | Part-1 |
| Stable across k-NN values (45.8-46.1%) | Cell 16 output | Part-1 |
| Stable across α values (45.7-45.9%) | Cell 18 output | Part-1 |
| 11.6% on ScreenSpot-Pro (GPT-4o: 0.9%) | Cell 27 output | Part-1 |
| 44 samples UVLA solves that Atlas misses | Complementarity output | VS_OSATLAS |
| 87.3% oracle accuracy | Complementarity output | VS_OSATLAS |
| Different failure modes (text vs icon) | Text/Icon table output | VS_OSATLAS |

---

## Models Used (All General-Purpose, Zero UI Training)

| Model | Parameters | Purpose in Pipeline | Equation |
|-------|-----------|-------------------|----------|
| CLIP ViT-B/32 | ~150M | Vision similarity scoring | Eq. 3 |
| SBERT MiniLM-L6-v2 | ~22M | Text similarity scoring | Eq. 4 |
| EasyOCR | ~5M | Text extraction for proposals | — |
| **Total** | **~170M** | **All frozen, general-purpose** | — |

**None of these models have ever seen UI training data.**

---

## Anonymity Statement

This repository contains no author-identifying information. All models and datasets used are publicly available. The HuggingFace token field is a placeholder — reviewers should use their own token.
