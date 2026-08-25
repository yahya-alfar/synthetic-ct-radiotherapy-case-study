# MRI → Synthetic CT for Radiotherapy

**Technical Case Study | Medical Imaging · Deep Learning · Applied AI**

Research conducted during **SURE 2026 at Alfaisal University**, within a medical-imaging research collaboration involving **King Faisal Specialist Hospital & Research Centre (KFSHRC)**.

> **Repository scope:** This repository documents non-sensitive methodology, experimental reasoning, and validation outcomes. Source code, clinical data, patient-level images, model weights, and private platform implementation details are not included.

---

## Project Overview

MRI provides strong soft-tissue contrast for radiotherapy planning, but unlike CT it does not directly provide the attenuation information required for conventional dose-calculation workflows.

This project investigated **pelvic MRI-to-Synthetic CT (sCT)** generation using deep learning.

My work covered the broader experimental pipeline rather than model training alone:

**Data review → MRI/CT pairing → quality control → cohort curation → frozen splits → fine-tuning → validation → failure analysis → model selection**

The goal was to improve sCT quality while maintaining a reproducible and validation-driven workflow.

---

## Project Highlights

- Fine-tuned a pretrained **Med2Transformer** for pelvic MRI-to-sCT generation
- Improved male validation MAE from **73.13 HU to 65.06 HU**
- Reduced female zero-shot validation MAE from **261.51 HU to 102.04 HU**
- Achieved **87.38 HU** female validation MAE after female-specific fine-tuning
- Evaluated models using **MAE, SSIM, PSNR, anatomical coverage, and tissue-level error analysis**
- Kept test cohorts locked during iterative model development and selection

> All numerical results shown here are **validation results unless explicitly stated otherwise**.

---

## My Role

My contributions focused on the technical and experimental side of the project:

- MRI/CT dataset review and cohort curation
- case-level quality-control decisions
- train / validation / test split design
- deep learning model fine-tuning
- controlled comparison of optimization strategies
- cross-cohort generalization analysis
- tissue-level and anatomical error analysis
- validation-driven model selection
- research workflow structuring and result review

---

## Problem Decomposition

I treated the task as more than a direct image-to-image translation problem.

Model performance depended on several interacting factors:

- MRI/CT pairing and registration quality
- anatomical and field-of-view coverage
- patient-level inclusion and exclusion criteria
- leakage-resistant dataset splitting
- training strategy
- cross-cohort generalization
- tissue-specific failure patterns
- evaluation discipline

This led to a workflow where **data quality and experimental design were addressed before model optimization**.

---

## Dataset & Cohort Design

### Male / Prostate Cohort

**54 reviewed → 47 included → 7 excluded**

| Split | Cases |
|---|---:|
| Train | 33 |
| Validation | 7 |
| Test | 7 |

Two additional training cases were later excluded following further quality review, resulting in an effective training cohort of **31 cases** for the selected refinement experiment.

### Female Pelvic Cohort

**44 reviewed → 28 included → 16 excluded**

| Split | Cases |
|---|---:|
| Train | 19 |
| Validation | 4 |
| Test | 5 |

The test cohorts remained locked during development and model selection.

---

## Data Quality Strategy

A case being available did not automatically make it suitable for model development.

The workflow separated:

**Discovery → Review → Quality Control → Human Decision → Inclusion → Experimental Split**

Quality review considered:

- MRI/CT correspondence
- registration quality
- anatomical coverage
- body / field-of-view coverage
- suitability for model development

This helped prevent poor-quality cases from silently affecting downstream experiments.

---

## Experimental Strategy

The project used a pretrained pelvic **Med2Transformer** as the starting point.

Experiments were organized as controlled iterations:

```text
Pretrained Pelvic Model
        ↓
V1 Fine-Tuning
        ↓
Bone-Aware Experiments
        ↓
V2-C Refinement
        ↓
Female Zero-Shot Evaluation
        ↓
Female-Specific Fine-Tuning
        ↓
Bone-Focused Follow-Up
```

The general reasoning pattern was:

**Observation → Hypothesis → Controlled Change → Validation → Decision**

---

## Male Cohort Results

| Experiment | Main Change | Validation MAE |
|---|---|---:|
| V1 | Initial fine-tuning | 73.13 HU |
| V2-A | Bone-aware sampling | 75.45 HU |
| V2-B | Bone-aware sampling + bone-weighted L1 | 75.84 HU |
| **V2-C** | Low-LR refinement + spacing-aware gradient objective | **65.06 HU** |

**V2-C** was retained as the validation-selected male model.

Compared with V1:

**73.13 HU → 65.06 HU**

Approximately **11% improvement in validation MAE**.

---

## Cross-Cohort Generalization

The male V2-C model was evaluated on the female validation cohort **before female-specific fine-tuning**.

| Model | Female Validation MAE |
|---|---:|
| Original pretrained pelvic model | 261.51 HU |
| **Male V2-C** | **102.04 HU** |

This corresponds to approximately a:

**61% reduction in female validation MAE**

without female-specific fine-tuning.

This made cross-cohort transfer and generalization an important part of the analysis rather than treating each cohort as an isolated problem.

---

## Female-Specific Fine-Tuning

The male V2-C model was used as the initialization point for female-specific fine-tuning.

| Metric | Validation Result |
|---|---:|
| **MAE** | **87.38 HU** |
| SSIM | 0.543 |
| PSNR | 19.14 |

Female-specific fine-tuning improved MAE further:

**102.04 HU → 87.38 HU**

---

## Failure Analysis

Several theoretically reasonable experiments did **not** improve validation performance:

- bone-aware sampling: **73.13 → 75.45 HU**
- bone-weighted L1: **75.84 HU**
- model interpolation: no tested ratio outperformed V2-C
- female bone-focused fine-tuning: **87.38 → 88.06 HU**

These experiments were not retained.

### Key Engineering Takeaway

> **Improving one difficult anatomical region in isolation does not necessarily improve overall model performance.**

Model-selection decisions therefore remained **validation-driven rather than assumption-driven**.

---

## Evaluation Integrity

The workflow separated model development from final testing:

```text
Training Data
     ↓
Validation Data
     ↓
Model Selection
     ↓
Locked Test Data
```

Hyperparameter changes, model comparisons, and refinement decisions were made using validation results.

The locked test cohorts were not used for iterative model selection.

---

## Research Workflow

```text
MRI / CT Data
      ↓
Data Discovery
      ↓
Pairing & Registration Review
      ↓
Quality Control
      ↓
Human Curation
      ↓
Approved Cohort
      ↓
Frozen Train / Validation / Test Splits
      ↓
Model Fine-Tuning
      ↓
Validation
      ↓
Anatomical & Tissue-Level Analysis
      ↓
Experiment Comparison
      ↓
Model Selection
```

The workflow was designed so that **data decisions, model experiments, and evaluation decisions remained logically separated and traceable**.

---

## Technical Documentation

For a deeper technical view of the project:

- [Experimental History](docs/experiments.md) — model-development sequence, hypotheses, configurations, outcomes, and model-selection decisions
- [Evaluation & Failure Analysis](docs/evaluation.md) — image-level metrics, tissue-level analysis, coverage review, and failure interpretation
- [Methodology](docs/methodology.md) — data curation, MRI/CT review, quality-control strategy, frozen splits, and research workflow design
- [Scientific References](docs/references.md) — selected literature informing the scientific context and evaluation framework

---

## Core Technical Areas

**Machine Learning:** Python · PyTorch · Med2Transformer · Fine-Tuning · Transfer Learning · Controlled Experimentation

**Medical Imaging:** MRI · CT · Synthetic CT · MRI/CT Pairing · Registration Review · Hounsfield Unit Analysis

**Evaluation:** MAE · SSIM · PSNR · Tissue-Level Error Analysis · Cross-Cohort Generalization · Dataset Quality Control

---

## What This Project Reinforced

- **Data quality is part of model engineering**
- **Reproducible splits are design decisions, not afterthoughts**
- **Global metrics can hide local anatomical failures**
- **Negative experiments can eliminate incorrect assumptions**
- **Model selection should follow evidence rather than attachment to an approach**

---

## Private Platform Work

As part of the broader research workflow, I also designed and developed an interactive medical-imaging research platform supporting areas such as **data discovery, MRI/CT review, quality control, human curation, experiment preparation, and model-result review**.

Its implementation, architecture, source code, and internal design details are not included while intellectual-property registration is in progress.

---

## Repository Scope & Privacy

This repository is a **technical case study rather than a source-code release**.

Not included:

- patient data or clinical images
- DICOM files or patient identifiers
- private research code
- trained model weights
- internal hospital or research infrastructure
- private platform implementation
- proprietary workflow or quality-control implementation details

The repository focuses on the **problem framing, experimental methodology, validation strategy, engineering reasoning, and non-sensitive technical outcomes** of the work.

---

## Scientific Context

Synthetic CT generation addresses a core challenge in MRI-based radiotherapy: MRI provides strong soft-tissue contrast, while CT traditionally provides the information used for radiotherapy dose calculation.

The project was informed by established literature on MRI-only radiotherapy, deep-learning-based synthetic CT generation, and modern sCT benchmarking.

### Selected Background References

- Schmidt MA, Payne GS. **Radiotherapy planning using MRI.** *Physics in Medicine & Biology*, 2015.
- Edmund JM, Nyholm T. **A review of substitute CT generation for MRI-only radiation therapy.** *Radiation Oncology*, 2017.
- Spadea MF, Maspero M, Zaffino P, Seco J. **Deep learning based synthetic-CT generation in radiotherapy and PET: A review.** *Medical Physics*, 2021.
- Thummerer A, et al. **SynthRAD2023 Grand Challenge dataset: Generating synthetic CT for radiotherapy.** *Medical Physics*, 2023.
- Huijben EMC, et al. **Generating synthetic computed tomography for radiotherapy: SynthRAD2023 challenge report.** *Medical Image Analysis*, 2024.

---

## Status

**Research phase completed for the work documented in this case study.**

Further non-sensitive technical material may be added where appropriate and where project, privacy, research, and intellectual-property restrictions allow.
