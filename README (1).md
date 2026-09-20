# ZEBRA-SCAN

## AI-Assisted Zebrafish Developmental Phenotype Screening

ZEBRA-SCAN is a multi-label deep-learning system for screening developmental abnormalities in zebrafish bright-field microscopy images.

The project combines a complete experimental machine-learning pipeline with a browser-based inference application. The final deployed system uses a frozen five-fold ResNet18 ensemble.

## Project Overview

The system predicts eight phenotypes:

1. Bent spine
2. Jaw malformation
3. Swim bladder absence
4. Yolk edema
5. Pericardial edema
6. Dead
7. Head hemorrhage
8. Unhatched embryo

The task is formulated as a multi-label classification problem because a single image can contain multiple phenotypes simultaneously.

## Dataset

The project uses a COCO-annotated zebrafish developmental phenotype dataset derived from the published 2023 toxicity-screening study.

- **1,328 images**
- **8,417 annotations**
- **16 COCO categories**

The dataset itself is not included in this repository.

## Experimental Design

Main hypothesis:

> Combining learned visual representations with explicit anatomical/morphometric information improves multi-label detection of zebrafish developmental abnormalities compared with an image-only model.

An independent split was constructed across all 1,328 images:

- Development set: **1,062 images**
- Locked test set: **266 images**

The development set used **5-fold multilabel-stratified cross-validation**. The locked test set was kept untouched until final evaluation.

## Models

### A0 — Scratch CNN

An image-only convolutional neural network trained from scratch and used as a baseline.

### A1 — Pretrained ResNet18

A pretrained ImageNet ResNet18 adapted for eight-label multilabel classification.

Five fold models were retained and combined into the final ensemble.

### B3 — Morphology Model

Explicit anatomical and morphometric features derived from COCO annotations, including:

- Area
- Area fraction
- Bounding-box dimensions
- Aspect ratio
- Centroid coordinates
- Perimeter
- Circularity
- Solidity
- Relative anatomical geometry
- Spine geometry and curvature features

Missing anatomy was treated as missing information rather than automatically interpreted as biological absence.

### Hybrid Model

A1 visual embeddings were combined with B3 morphometric features and evaluated using nested cross-validation.

## Development Results

| Model | Macro-F1 | Micro-F1 | Macro PR-AUC |
|---|---:|---:|---:|
| A0 Scratch CNN | 0.7331 | 0.7619 | 0.7913 |
| B3 Morphology | 0.8444 | 0.9621 | 0.8669 |
| A1 ResNet18 | **0.8970** | 0.9139 | **0.9502** |
| Hybrid | 0.8537 | 0.9000 | 0.9070 |

The pretrained A1 model achieved the strongest overall development performance by Macro-F1 and Macro PR-AUC. The hybrid model did not improve aggregate performance over A1 under the evaluated fusion setup.

## Frozen Decision Thresholds

The final A1 ensemble uses phenotype-specific thresholds optimized from held-out out-of-fold development predictions.

| Phenotype | Threshold |
|---|---:|
| bent_spine | 0.45 |
| jaw_malformation | 0.39 |
| swim_bladder_absence | 0.65 |
| yolk_edema | 0.60 |
| pericardial_edema | 0.68 |
| dead | 0.85 |
| head_hemorrhage | 0.90 |
| unhatched_embryo | 0.24 |

These thresholds were frozen before locked-test evaluation.

## Locked Test Performance

The final five-fold A1 ensemble was evaluated on the locked 266-image test set.

| Metric | Score |
|---|---:|
| Macro-F1 | **0.9048** |
| Micro-F1 | **0.9317** |
| Macro Precision | **0.9271** |
| Macro Recall | **0.9006** |
| Macro PR-AUC | **0.9734** |

### Per-Phenotype F1

| Phenotype | F1 |
|---|---:|
| bent_spine | 0.8531 |
| jaw_malformation | 0.9167 |
| swim_bladder_absence | 0.9778 |
| yolk_edema | 0.9220 |
| pericardial_edema | 0.9498 |
| dead | 0.7273 |
| head_hemorrhage | 0.9231 |
| unhatched_embryo | 0.9691 |

## Failure Analysis

The locked test set contained:

- **89 phenotype-level errors**
- **70 images containing at least one error**
- **55 false positives**
- **34 false negatives**

Observed failure mechanisms included:

1. **Morphological ambiguity** — visually similar structures can lead to phenotype confusion.
2. **Weak or subtle phenotype signal** — subtle abnormalities can be difficult to distinguish.
3. **Salient visual confounding** — visually prominent regions can drive predictions even when the corresponding annotation is negative.

Grad-CAM was used to investigate representative failures. Grad-CAM is treated as an explanatory visualization rather than proof of biological reasoning.

## Browser Deployment

The deployment is designed as a browser-side application:

```text
Image Upload
     |
     v
224 × 224 preprocessing
     |
     v
ResNet18 Fold 0 ─┐
ResNet18 Fold 1  │
ResNet18 Fold 2  ├── Ensemble average
ResNet18 Fold 3  │
ResNet18 Fold 4 ─┘
     |
     v
8 phenotype probabilities
     |
     v
Frozen phenotype-specific thresholds
     |
     v
Predicted phenotypes
     |
     v
Grad-CAM visualization
```

The browser application uses:

- ONNX models
- ONNX Runtime Web
- JavaScript
- HTML/CSS
- Browser-side inference

No training data is required for inference and no model retraining occurs during deployment.

## Repository Structure

```text
ZEBRA-SCAN/
│
├── README.md
├── index.html
├── classes.json
├── classifier_weights.json
│
├── models/
│   ├── A1_resnet18_fold0.onnx
│   ├── A1_resnet18_fold1.onnx
│   ├── A1_resnet18_fold2.onnx
│   ├── A1_resnet18_fold3.onnx
│   └── A1_resnet18_fold4.onnx
│
└── notebooks/
    └── ZEBRA_SCAN_full_analysis.ipynb
```

## Reproducibility

The research notebook documents:

- Dataset construction
- Exploratory data analysis
- Data preprocessing
- Independent splitting
- Morphometric feature construction
- CNN baselines
- ResNet18 training
- Cross-validation
- Hybrid modeling
- Threshold optimization
- Locked-test evaluation
- Failure analysis

The deployed browser application uses the resulting frozen A1 models and does not retrain them.

## Scientific Caveats

The model is an AI screening system rather than a diagnostic system.

Model probabilities should not be interpreted as biological certainty.

The morphology model uses annotated anatomical structures and therefore represents an experimental/oracle feature setting rather than a fully image-only deployment pathway.

Annotation availability was treated carefully because missing anatomical annotations can reflect the annotation protocol rather than biological absence.

Grad-CAM visualizations indicate spatial contributions to model outputs and should not be interpreted as biological ground-truth lesion maps.

## Citation

The underlying dataset is derived from:

> Deep Learning-Enabled Morphometric Analysis for Toxicity Screening Using Zebrafish Larvae. ACS, 2023.

Please cite the original study when using the dataset or derived results.

## Disclaimer

ZEBRA-SCAN is a research and educational machine-learning project.

It is intended for experimental screening and computational research and is **not a validated diagnostic or toxicological decision-making system**.
