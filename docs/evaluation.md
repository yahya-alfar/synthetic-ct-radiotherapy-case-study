# Evaluation & Failure Analysis

This document expands on the evaluation strategy used in the MRI-to-Synthetic-CT project.

For the experimental sequence itself, see [Experimental History](experiments.md).

> **Evaluation note:** Results described here are validation results unless explicitly stated otherwise. Locked test cohorts were not used for iterative model selection.

---

## Evaluation Philosophy

Synthetic-CT quality cannot be understood from a single global metric.

A model may improve overall image error while still performing poorly in clinically important or anatomically difficult regions.

For that reason, evaluation was approached at multiple levels:

**Global image metrics → tissue-level analysis → anatomical coverage review → failure-case inspection → model comparison**

The objective was not simply to identify the model with the lowest number, but to understand **where the improvement occurred and where limitations remained**.

---

# 1. Image-Level Metrics

The primary quantitative metrics were:

### MAE — Mean Absolute Error

MAE measures the average absolute difference in Hounsfield Units between the generated synthetic CT and the reference CT.

Lower values indicate lower average HU error.

### SSIM — Structural Similarity Index

SSIM was used to assess structural similarity between generated and reference images.

Higher values indicate stronger structural agreement.

### PSNR — Peak Signal-to-Noise Ratio

PSNR provides another measure of image reconstruction quality.

Higher values generally indicate lower reconstruction error.

---

# 2. Male Validation Selection

The selected male model was **V2-C**.

| Model | Validation MAE |
|---|---:|
| V1 | 73.13 HU |
| V2-A | 75.45 HU |
| V2-B | 75.84 HU |
| **V2-C** | **65.06 HU** |

V2-C improved validation MAE by approximately **11%** relative to V1.

The model was therefore retained based on validation performance rather than on the perceived attractiveness of any particular optimization strategy.

---

# 3. Female Zero-Shot Evaluation

Before female-specific fine-tuning, the selected male V2-C model was evaluated directly on the female validation cohort.

| Model | MAE | SSIM | PSNR |
|---|---:|---:|---:|
| Original pretrained pelvic model | 261.51 HU | 0.432 | 17.04 |
| **Male V2-C** | **102.04 HU** | **0.493** | **18.94** |

The male-adapted model reduced female validation MAE by approximately:

**61%**

relative to the original pretrained model.

This result was important because the improvement occurred **without female-specific training**.

It suggested that the male fine-tuning stage produced representations that transferred substantially better to the female pelvic cohort.

---

# 4. Female-Specific Fine-Tuning

Female FT1 was initialized from the selected male V2-C model.

The best validation checkpoint produced:

| Metric | Result |
|---|---:|
| **MAE** | **87.38 HU** |
| SSIM | 0.543 |
| PSNR | 19.14 |

Compared with the zero-shot V2-C result:

**102.04 HU → 87.38 HU**

This confirmed that female-specific adaptation provided additional improvement after cross-cohort transfer.

---

# 5. Tissue-Level Evaluation

Global MAE alone did not explain where the model improved.

The evaluation was therefore extended to tissue-specific regions.

Selected tissue-level changes after female-specific fine-tuning were:

| Tissue / Region | Before FT1 | After FT1 | Interpretation |
|---|---:|---:|---|
| Fat | 86.0 HU | **61.9 HU** | Strong improvement |
| Soft Tissue | 53.3 HU | **48.4 HU** | Moderate improvement |
| Bone | 489.4 HU | 483.9 HU | Limited improvement |
| Air / Gas | 713.3 HU | 697.5 HU | Limited improvement |

The largest improvement occurred in **fat**.

Soft tissue also improved, while bone and air / gas remained substantially more difficult.

---

# 6. Why Tissue-Level Analysis Mattered

The tissue-level results showed that:

- a lower global MAE did not imply equal improvement across all anatomy
- common soft-tissue regions responded better to fine-tuning
- high-HU bone remained difficult
- air / gas regions continued to produce large errors

This changed how model quality was interpreted.

Rather than asking only:

> Did the global MAE improve?

the evaluation also asked:

> **Where did it improve, and what remained difficult?**

---

# 7. Bone-Focused Evaluation

Bone remained one of the most difficult regions throughout the experiments.

A bone-focused evaluation was therefore used to understand whether strategies that explicitly emphasized bone were actually improving high-HU anatomy.

The results showed an important trade-off:

Bone-oriented training strategies could increase attention to difficult structures without necessarily improving global synthetic-CT performance.

This was observed in:

- bone-aware sampling
- bone-weighted L1
- female BoneLite fine-tuning

None of these strategies outperformed the validation-selected models on global MAE.

The result reinforced the need to evaluate both:

**regional behavior and whole-image performance**

rather than optimizing either one in isolation.

---

# 8. Anatomical Coverage Analysis

Model error was also interpreted in the context of MRI anatomical coverage.

One female validation case had markedly lower MRI coverage than the rest of the validation cohort.

Its supported-body coverage was approximately:

**56%**

while the other validation cases had substantially greater anatomical coverage.

This mattered because incomplete or limited input coverage can affect the apparent performance of an image-synthesis model.

---

# 9. Failure-Case Interpretation

A high-error case does not automatically mean:

> the neural network failed.

Possible contributing factors can include:

- incomplete MRI field of view
- anatomical coverage differences
- registration limitations
- unusual anatomy
- image-quality variation
- genuine model limitations

For this reason, failure cases were reviewed alongside aggregate metrics rather than being interpreted from MAE alone.

---

# 10. Aggregate vs. Local Performance

The evaluation strategy separated two different questions.

### Global Question

> How close is the generated sCT to the reference CT overall?

Measured using:

- MAE
- SSIM
- PSNR

### Local / Anatomical Question

> Where is the model accurate, and where does it remain unreliable?

Investigated using:

- tissue-level HU error
- bone-focused analysis
- coverage review
- case-level failure inspection

A model could improve the first question without fully solving the second.

---

# 11. Evaluation-Driven Model Selection

Model selection followed a simple principle:

> **A new experiment had to demonstrate measurable improvement under comparable validation conditions.**

This prevented theoretically attractive ideas from being retained when they did not produce better evidence.

Examples included:

| Experiment | Outcome |
|---|---|
| Bone-aware sampling | Global MAE worsened |
| Bone-weighted L1 | Global MAE worsened |
| Model interpolation | No tested ratio improved V2-C |
| BoneLite female fine-tuning | Slightly worsened global MAE |

The selected models remained:

### Male
**V2-C — 65.06 HU validation MAE**

### Female
**FT1 — 87.38 HU validation MAE**

---

# 12. Evaluation Integrity

The development process maintained the following separation:

```text
Training
   ↓
Validation
   ↓
Model Selection
   ↓
Locked Test
```

Validation data was used for:

- checkpoint comparison
- model selection
- experiment comparison
- refinement decisions

Locked test cohorts were reserved outside the iterative development loop.

This distinction was important to reduce optimistic bias from repeated evaluation on test data.

---

# 13. Main Evaluation Lessons

### 1. MAE is useful but incomplete

A lower global MAE does not guarantee improvement across every tissue type.

### 2. Anatomy matters

Model quality needs to be interpreted in the context of anatomical regions and input coverage.

### 3. Failure cases require context

Poor performance can result from data limitations as well as model limitations.

### 4. Regional optimization can introduce trade-offs

Improving attention to one tissue class does not guarantee better whole-image reconstruction.

### 5. Validation should drive selection

Models were retained based on comparable validation evidence rather than theoretical preference.

---

## Related Documentation

- [Main Case Study](../README.md)
- [Experimental History](experiments.md)
- Methodology details — `methodology.md` *(planned)*
- Scientific references — `references.md` *(planned)*

---

## Repository Scope

This document contains only non-sensitive aggregate evaluation information.

It does not include:

- patient images
- DICOM data
- patient identifiers
- private evaluation code
- internal platform implementation
- model weights
- proprietary clinical or research infrastructure
