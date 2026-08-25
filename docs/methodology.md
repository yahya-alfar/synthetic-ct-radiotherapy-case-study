# Methodology

This document describes the methodology used to structure the MRI-to-Synthetic-CT research workflow, with emphasis on **data curation, quality control, reproducible cohort design, experimental separation, and validation discipline**.

For the main project summary, see the [Main Case Study](../README.md).

For model-development details, see [Experimental History](experiments.md).

For quantitative evaluation, see [Evaluation & Failure Analysis](evaluation.md).

> **Scope:** This document describes non-sensitive methodological principles and aggregate workflow decisions. Patient-level data, private implementation details, and proprietary platform logic are not included.

---

## 1. Methodological Principle

The project was approached as an end-to-end research workflow rather than only a model-training task.

The core sequence was:

```text
Available MRI / CT Data
        ↓
Case Discovery
        ↓
Pair Verification
        ↓
Registration & Coverage Review
        ↓
Quality-Control Decision
        ↓
Cohort Inclusion / Exclusion
        ↓
Frozen Patient-Level Splits
        ↓
Model Development
        ↓
Validation
        ↓
Failure Analysis
        ↓
Model Selection
```

The underlying principle was:

> **Decisions about data quality should be made before model results are allowed to influence those decisions.**

This helped keep data curation and model optimization logically separate.

---

# 2. Data Discovery

The first stage was identifying available MRI and CT data that could potentially form usable paired cases.

Availability alone was not considered sufficient for inclusion.

Each candidate case had to pass subsequent review before entering the experimental cohort.

This distinction was important because a dataset can technically contain both modalities while still being unsuitable for supervised image translation.

---

# 3. MRI / CT Pair Review

The project required paired MRI and CT data representing sufficiently corresponding anatomy.

Review therefore considered whether:

- both modalities were available
- the images represented the expected anatomical region
- MRI and CT could be meaningfully paired
- major anatomical mismatch was present
- registration quality was sufficient for the intended experiment
- MRI coverage supported meaningful comparison with CT

Cases that failed these checks were not automatically carried into model development.

---

# 4. Registration Quality

MRI-to-CT synthesis depends heavily on spatial correspondence between source and target images.

If anatomy is poorly aligned, the model can be penalized for differences caused by registration rather than synthesis quality.

Registration review was therefore treated as a methodological requirement rather than a preprocessing detail.

The review focused on whether paired volumes showed sufficiently consistent anatomy for downstream learning and evaluation.

Potential issues included:

- anatomical displacement
- differences in body positioning
- incomplete overlap
- inconsistent field of view
- local mismatch between MRI and CT

The objective was not to assume that every paired scan was automatically a valid supervised training pair.

---

# 5. Anatomical Coverage

Coverage was treated separately from registration quality.

A pair could be reasonably aligned while still containing insufficient MRI coverage relative to the CT reference.

This became particularly important during female-cohort evaluation, where one validation case showed substantially reduced supported-body MRI coverage.

The methodological implication was:

> **Input completeness can influence apparent model error independently of model capability.**

For this reason, anatomical coverage was included in failure-case interpretation.

---

# 6. Case-Level Quality Control

Candidate cases were reviewed using an explicit acceptance / exclusion process.

The conceptual QC flow was:

```text
Candidate Case
      ↓
MRI Available?
      ↓
CT Available?
      ↓
Usable Pair?
      ↓
Registration Acceptable?
      ↓
Coverage Acceptable?
      ↓
Anatomically Suitable?
      ↓
Include / Exclude
```

The actual project workflow combined automated evidence with human review where appropriate.

This was intentional.

Not every medical-imaging quality decision can be reduced safely to a single threshold.

---

# 7. Human-in-the-Loop Review

The methodology separated two kinds of decisions:

### Automated or Computable Evidence

Examples included measurable properties of image coverage, image dimensions, spatial information, and other case-level indicators.

### Human Research Review

Used when interpretation required visual or contextual judgment.

This allowed the workflow to avoid two extremes:

- relying entirely on manual inspection
- treating numerical thresholds as automatically sufficient

The goal was to support **structured human judgment with traceable evidence**.

---

# 8. Cohort Construction

After review, accepted cases were separated from excluded cases before model development.

## Male / Prostate Cohort

Initial review:

**54 cases → 47 included → 7 excluded**

Frozen split:

| Split | Cases |
|---|---:|
| Train | 33 |
| Validation | 7 |
| Test | 7 |

Two training cases were later removed after additional quality review, giving an effective training cohort of:

**31 cases**

for the selected refinement experiment.

---

## Female Pelvic Cohort

Initial review:

**44 cases → 28 included → 16 excluded**

Frozen split:

| Split | Cases |
|---|---:|
| Train | 19 |
| Validation | 4 |
| Test | 5 |

The validation cohort was used for iterative model comparison.

The test cohort remained outside the iterative development loop.

---

# 9. Why Patient-Level Frozen Splits Mattered

The split design was treated as a methodological decision rather than a convenience.

The goals were to:

- prevent patient leakage across splits
- make model comparisons reproducible
- prevent later experiments from receiving easier or harder validation sets
- maintain a stable basis for comparing optimization strategies
- protect final test evaluation from iterative tuning

The resulting structure was:

```text
Accepted Patients
      ↓
Patient-Level Split
      ↓
┌──────────┬──────────────┬──────────┐
│ Training │ Validation   │ Test     │
│          │              │          │
│ Develop  │ Select       │ Final    │
└──────────┴──────────────┴──────────┘
```

Once established, these splits were treated as frozen.

---

# 10. Separation of Development and Testing

The experimental methodology distinguished three different purposes.

### Training Data

Used to optimize model parameters.

### Validation Data

Used to:

- compare experiments
- select checkpoints
- assess proposed modifications
- identify candidate models
- guide further development

### Test Data

Reserved for final evaluation rather than iterative decision-making.

This distinction was maintained to reduce optimistic bias from repeatedly evaluating against the final test cohort.

---

# 11. Baseline-First Experimentation

Before introducing specialized losses or sampling strategies, a project-specific baseline was established.

This became **V1**.

Subsequent experiments were evaluated relative to that baseline.

The reasoning structure was:

```text
Establish Baseline
      ↓
Identify Limitation
      ↓
Form Hypothesis
      ↓
Modify One Main Component
      ↓
Evaluate on Same Validation Set
      ↓
Keep / Reject
```

This reduced the risk of making multiple simultaneous changes and then being unable to identify what caused the observed result.

---

# 12. Controlled Experimentation

The main male-cohort development sequence included:

- baseline fine-tuning
- bone-aware sampling
- bone-weighted loss
- conservative low-learning-rate refinement
- model interpolation

The female sequence included:

- zero-shot evaluation of the male-adapted model
- female-specific fine-tuning
- bone-focused follow-up

Each experiment attempted to answer a specific question rather than simply generate another training run.

Examples:

> Does increasing exposure to bone-rich regions improve validation performance?

> Does explicit bone weighting improve high-HU prediction without harming global MAE?

> Does the male-adapted representation transfer to female anatomy?

> Does female-specific fine-tuning add value after zero-shot transfer?

This question-driven structure made negative experiments useful rather than disposable.

---

# 13. Validation-Driven Decision Making

A methodological rule throughout development was:

> **A theoretically attractive change was not sufficient reason to retain an experiment.**

The experiment had to improve or meaningfully clarify behavior under comparable validation conditions.

For example:

- bone-aware sampling was reasonable conceptually
- bone-weighted loss was also reasonable conceptually
- bone-focused female refinement targeted a known difficult region

However, none were retained because they did not improve the selected validation objective.

This prevented experimental preference from overriding observed evidence.

---

# 14. Cross-Cohort Evaluation

After male fine-tuning, the selected male model was evaluated on the female validation cohort before any female-specific training.

This served a different methodological purpose from standard validation.

The question was not:

> How well does the model fit another split of the same training population?

Instead:

> **Has the adapted model learned representations that transfer meaningfully to another pelvic cohort?**

This evaluation exposed a strong reduction in female zero-shot MAE relative to the original pretrained model and motivated the next stage of female-specific adaptation.

---

# 15. Multi-Level Evaluation

Model evaluation was organized at several levels.

## Global Image Level

- MAE
- SSIM
- PSNR

## Tissue Level

- fat
- soft tissue
- bone
- air / gas

## Case Level

- anatomical coverage
- difficult cases
- possible data-quality limitations
- failure inspection

This prevented a single aggregate score from becoming the only interpretation of model quality.

---

# 16. Failure Analysis as Part of Methodology

Failure analysis was treated as part of model development rather than as a final reporting step.

When performance was worse than expected, the workflow asked:

```text
Was the hypothesis wrong?
        ↓
Was optimization ineffective?
        ↓
Was the tissue objective too narrow?
        ↓
Was input coverage limited?
        ↓
Was registration a possible contributor?
        ↓
Is this a global or localized failure?
```

This was especially important for difficult regions such as:

- bone
- air / gas
- limited-coverage anatomy

The objective was to distinguish **model failure from data or evaluation limitations whenever possible**.

---

# 17. Traceability

A major workflow goal was to preserve traceability between:

- discovered cases
- inclusion / exclusion decisions
- cohort membership
- frozen splits
- experiments
- validation outputs
- selected models

The broader research workflow was therefore structured so that each later stage could be connected back to earlier data decisions.

Conceptually:

```text
Case
 ↓
QC Decision
 ↓
Cohort
 ↓
Split
 ↓
Experiment
 ↓
Evaluation
 ↓
Selection Decision
```

This helps make experimental outcomes easier to interpret and reproduce.

---

# 18. Research Platform Support

To support the broader workflow, I also designed and developed an interactive research platform around parts of this process.

The platform was intended to make research stages such as:

- data discovery
- MRI/CT review
- quality control
- human curation
- experiment preparation
- result review

more visible and structured.

The methodological principle behind the platform was:

> **Expose the reasoning and state of the workflow rather than hiding important research decisions inside backend scripts.**

Implementation details, architecture, internal data structures, and source code are not included while intellectual-property registration is in progress.

---

# 19. Methodology Summary

The overall methodology can be summarized as:

```text
1. Discover candidate data
        ↓
2. Verify MRI / CT pairing
        ↓
3. Review registration and coverage
        ↓
4. Apply structured QC
        ↓
5. Curate accepted cohorts
        ↓
6. Freeze patient-level splits
        ↓
7. Establish baseline
        ↓
8. Run question-driven experiments
        ↓
9. Compare on validation data
        ↓
10. Inspect tissue and failure patterns
        ↓
11. Select model using evidence
        ↓
12. Preserve locked test data for final evaluation
```

The core idea was that **reliable machine learning depends on the structure surrounding the model as much as on the model itself**.

---

## Related Documentation

- [Main Case Study](../README.md)
- [Experimental History](experiments.md)
- [Evaluation & Failure Analysis](evaluation.md)
- Scientific references — `references.md` *(planned)*

---

## Repository Scope

This methodological description contains only non-sensitive project information.

It does not disclose:

- patient-level data
- clinical images
- DICOM files
- private source code
- private database design
- proprietary quality-control implementation
- internal institutional infrastructure
- private platform architecture
