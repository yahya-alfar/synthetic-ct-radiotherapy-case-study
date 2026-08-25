# Experimental History

This document provides a more detailed record of the model-development experiments summarized in the main [README](../README.md).

The purpose is not to list every training run, but to document the main experimental decisions, what motivated them, what changed, what happened, and why a model or strategy was retained or rejected.

> **Evaluation note:** Results below are validation results unless explicitly stated otherwise. Locked test cohorts were not used for iterative model selection.

---

## Experimental Philosophy

The project followed a controlled iteration pattern:

**Observation → Hypothesis → Controlled Change → Validation → Decision**

The objective was to avoid changing multiple components simultaneously whenever possible.

This made it easier to determine whether an observed change in performance was plausibly connected to the experimental modification being tested.

---

# 1. Male Pelvic Development

## Starting Point

The project began from a pretrained pelvic **Med2Transformer** model.

The initial objective was to adapt the pretrained representation to the curated male / prostate cohort while establishing a reproducible validation baseline.

---

## V1 — Initial Fine-Tuning

### Objective

Establish a strong project-specific baseline through fine-tuning of the pretrained pelvic model.

### Result

**Validation MAE: 73.13 HU**

V1 became the baseline against which subsequent male-cohort experiments were compared.

### Decision

**Retained as baseline**

The next experiments investigated whether explicitly emphasizing difficult bone regions could improve performance further.

---

# 2. Bone-Aware Experiments

Bone remained one of the more difficult tissue regions because errors in high-HU structures can be substantially larger than errors in common soft-tissue regions.

This motivated experiments that deliberately increased the model's exposure or optimization emphasis toward bone.

---

## V2-A — Bone-Aware Sampling

### Hypothesis

If bone-rich regions were underrepresented during stochastic training, increasing their sampling probability might improve learning in difficult high-HU anatomy.

### Change

Training was modified to increase the probability of sampling bone-containing regions.

### Result

| Model | Validation MAE |
|---|---:|
| V1 | 73.13 HU |
| V2-A | 75.45 HU |

### Observation

Validation MAE worsened:

**73.13 HU → 75.45 HU**

### Decision

**Rejected**

The result suggested that increasing exposure to bone-rich samples alone did not improve global synthetic-CT accuracy.

---

## V2-B — Bone-Aware Sampling + Bone-Weighted L1

### Hypothesis

Sampling alone may not have provided a strong enough learning signal for high-HU structures.

A second experiment therefore added additional loss weighting to bone regions.

### Change

- bone-aware sampling retained
- bone-weighted L1 introduced

### Result

**Validation MAE: 75.84 HU**

### Observation

Performance remained worse than V1.

| Model | Validation MAE |
|---|---:|
| V1 | 73.13 HU |
| V2-A | 75.45 HU |
| V2-B | 75.84 HU |

### Decision

**Rejected**

The experiment reinforced that improving a difficult anatomical class in isolation does not guarantee improvement in the global objective.

---

# 3. V2-C — Low-Learning-Rate Refinement

After the bone-focused strategies failed to improve validation performance, the optimization strategy shifted away from increasing bone emphasis and toward more conservative refinement of the existing V1 solution.

## Objective

Improve the V1 model without aggressively altering the learned representation.

## Main Configuration

- initialized from **V1**
- learning rate: **5e-6**
- training duration: **12 epochs**
- masked normalized L1 objective
- light spacing-aware 3D gradient component
- validation-driven checkpoint selection

## Result

**Validation MAE: 65.06 HU**

Comparison:

| Model | Validation MAE |
|---|---:|
| V1 | 73.13 HU |
| V2-A | 75.45 HU |
| V2-B | 75.84 HU |
| **V2-C** | **65.06 HU** |

Relative improvement over V1:

**~11%**

### Decision

**Retained as the male validation-selected model**

V2-C produced the strongest male validation performance among the evaluated strategies.

---

# 4. Model Interpolation Experiments

After V2-C emerged as the strongest candidate, interpolation between candidate model states was explored to test whether a blended solution could outperform either model independently.

## Tested Ratios

Representative interpolation ratios included:

- 10 / 90
- 25 / 75
- 50 / 50

## Result

None of the tested interpolation configurations improved validation MAE relative to V2-C.

### Decision

**Rejected**

The selected V2-C checkpoint remained the male validation champion.

---

# 5. Female Zero-Shot Generalization

The next question was whether the male-adapted model had learned transferable pelvic representations rather than only cohort-specific behavior.

The female validation cohort was therefore evaluated **before any female-specific fine-tuning**.

## Comparison

| Model | Female Validation MAE | SSIM | PSNR |
|---|---:|---:|---:|
| Original pretrained pelvic model | 261.51 HU | 0.432 | 17.04 |
| **Male V2-C** | **102.04 HU** | **0.493** | **18.94** |

### Result

The male V2-C model reduced female validation MAE by approximately:

**61%**

relative to the original pretrained pelvic model.

### Interpretation

This was an important result because the improvement occurred **without female-specific training**.

It suggested that the male fine-tuning stage learned pelvic representations that transferred substantially better to the female cohort than the original pretrained model.

### Decision

Use **Male V2-C** as the initialization point for female-specific fine-tuning.

---

# 6. Female FT1 — Female-Specific Fine-Tuning

## Objective

Adapt the transferable male V2-C representation to the female pelvic cohort.

## Main Configuration

- initialized from **Male V2-C**
- epochs: **12**
- learning rate: **5e-6**
- masked normalized L1
- light spacing-aware gradient objective
- adversarial training retained

## Best Validation Result

Best checkpoint occurred at **epoch 12**.

| Metric | Result |
|---|---:|
| **MAE** | **87.38 HU** |
| SSIM | 0.543 |
| PSNR | 19.14 |

Improvement over female zero-shot V2-C:

**102.04 HU → 87.38 HU**

### Decision

**Retained as the female validation-selected model**

---

# 7. Female Tissue-Level Analysis

Global MAE improved after FT1, but the next question was:

> Where did that improvement actually occur?

Tissue-level evaluation was used to separate improvements in common soft tissue from persistent difficult regions.

| Tissue / Region | Before FT1 | After FT1 |
|---|---:|---:|
| Fat | 86.0 HU | **61.9 HU** |
| Soft Tissue | 53.3 HU | **48.4 HU** |
| Bone | 489.4 HU | 483.9 HU |
| Air / Gas | 713.3 HU | 697.5 HU |

## Interpretation

The largest relative improvement occurred in **fat**.

Soft tissue also improved.

Bone and air / gas remained substantially more difficult and showed only limited improvement.

This motivated one additional experiment focused specifically on bone.

---

# 8. FT2 — BoneLite

## Motivation

Although FT1 improved global validation performance, bone error remained high.

The objective was to determine whether a light bone-specific emphasis could improve high-HU prediction without substantially harming global performance.

## Main Configuration

- initialized from **FT1**
- epochs: **6**
- learning rate: **1.5e-6**
- bone threshold: **≥150 HU**
- bone weight: **1.5×**
- normalized weighted L1 objective

## Result

| Model | Validation MAE |
|---|---:|
| **FT1** | **87.38 HU** |
| FT2 BoneLite | 88.06 HU |

### Observation

BoneLite slightly worsened global validation MAE:

**87.38 HU → 88.06 HU**

### Decision

**Rejected**

FT1 remained the female validation champion.

---

# 9. Coverage-Related Failure Analysis

One female validation case showed substantially lower MRI anatomical coverage than the other cases.

This provided an important reminder that model error can arise from:

- model limitations
- registration limitations
- incomplete anatomical coverage
- unusual case characteristics
- differences in input quality

rather than network behavior alone.

For this reason, aggregate metrics were interpreted alongside case-level review.

---

# 10. Experiment Summary

| Stage | Experiment | Validation Outcome | Decision |
|---|---|---:|---|
| Male | V1 | 73.13 HU | Baseline |
| Male | V2-A Bone Sampling | 75.45 HU | Reject |
| Male | V2-B Bone Weighted | 75.84 HU | Reject |
| Male | **V2-C** | **65.06 HU** | **Select** |
| Female | Original Pretrained Zero-Shot | 261.51 HU | Reference |
| Female | Male V2-C Zero-Shot | 102.04 HU | Transfer baseline |
| Female | **FT1** | **87.38 HU** | **Select** |
| Female | FT2 BoneLite | 88.06 HU | Reject |

---

# 11. Main Experimental Lessons

## 1. A plausible hypothesis can still be wrong

Bone-aware sampling and bone weighting were reasonable ideas, but both worsened global validation performance.

The experiments were therefore rejected rather than defended based on intuition.

---

## 2. Conservative refinement can outperform aggressive reweighting

The strongest male result came from low-learning-rate refinement of an already useful representation rather than from explicit bone-focused optimization.

---

## 3. Generalization deserves its own experiment

The large reduction in female zero-shot error after male fine-tuning showed that model evaluation should not stop at within-cohort validation.

Cross-cohort transfer exposed behavior that would otherwise have remained hidden.

---

## 4. Global metrics are incomplete

FT1 improved global MAE substantially, but tissue-level analysis showed that different anatomical regions improved by very different amounts.

---

## 5. Negative experiments are useful evidence

V2-A, V2-B, interpolation, and BoneLite were not failures in the sense of wasted work.

Each eliminated a hypothesis and narrowed the search space.

---

# 12. Model-Selection Principle

The final selection strategy was simple:

> **Keep an experiment only when it improves performance under comparable validation conditions.**

The selected models were therefore:

### Male Champion
**V2-C — 65.06 HU validation MAE**

### Female Champion
**FT1 — 87.38 HU validation MAE**

The locked test cohorts were kept separate from iterative development and model selection.

---

## Related Documentation

- [Main Case Study](../README.md)
- Evaluation details — `evaluation.md` *(planned)*
- Methodology details — `methodology.md` *(planned)*
- Scientific references — `references.md` *(planned)*

---

## Repository Scope

This document contains only non-sensitive experimental summaries.

It does **not** include:

- patient data
- clinical images
- private model code
- trained weights
- private research-platform implementation
- internal institutional infrastructure
- proprietary workflow details
