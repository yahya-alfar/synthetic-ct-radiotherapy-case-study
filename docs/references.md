# Scientific References

This document lists selected references that informed the scientific context, model-selection discussions, and evaluation framework of the MRI-to-Synthetic-CT project.

The list is intentionally selective rather than exhaustive.

---

## 1. MRI in Radiotherapy Planning

### Schmidt & Payne — Radiotherapy Planning Using MRI

Schmidt MA, Payne GS. **Radiotherapy planning using MRI.**  
*Physics in Medicine & Biology.* 2015;60(22):R323–R361.

**DOI:**  
https://doi.org/10.1088/0031-9155/60/22/R323

### Relevance

This review provides important background on the role of MRI in radiotherapy planning, including:

- superior soft-tissue contrast
- geometric accuracy requirements
- patient positioning
- motion considerations
- the need to estimate electron-density information for dose calculation

It provides part of the clinical motivation for MRI-only and MRI-driven radiotherapy workflows.

---

## 2. MRI-Only Radiotherapy & Synthetic CT

### Edmund & Nyholm — Substitute CT Generation

Edmund JM, Nyholm T. **A review of substitute CT generation for MRI-only radiation therapy.**  
*Radiation Oncology.* 2017;12:28.

**DOI:**  
https://doi.org/10.1186/s13014-016-0747-y

### Relevance

This review summarizes approaches for generating CT-equivalent information from MRI for MRI-only radiotherapy.

Topics relevant to this project include:

- electron-density estimation
- synthetic / substitute CT generation
- geometric and dosimetric evaluation
- MRI-only treatment-planning workflows
- benchmarking challenges across different methods

---

## 3. Deep Learning for Synthetic CT

### Spadea et al. — Deep Learning-Based Synthetic CT Review

Spadea MF, Maspero M, Zaffino P, Seco J.  
**Deep learning based synthetic-CT generation in radiotherapy and PET: A review.**  
*Medical Physics.* 2021;48:6537–6566.

**DOI:**  
https://doi.org/10.1002/mp.15150

### Relevance

This review provides a broad overview of deep-learning approaches to synthetic CT generation.

It discusses:

- CNN-based approaches
- GAN-based approaches
- paired and unpaired learning
- MRI-to-CT synthesis
- CBCT-to-CT synthesis
- image-quality metrics
- clinical evaluation considerations

It also highlights the diversity of architectures and evaluation protocols used across the sCT literature.

---

## 4. SynthRAD2023 Dataset

### Thummerer et al. — SynthRAD2023 Grand Challenge Dataset

Thummerer A, van der Bijl E, Galapon A Jr, et al.  
**SynthRAD2023 Grand Challenge dataset: Generating synthetic CT for radiotherapy.**  
*Medical Physics.* 2023;50:4664–4674.

**DOI:**  
https://doi.org/10.1002/mp.16529

### Relevance

The SynthRAD2023 dataset paper provides an important modern reference for:

- MRI-to-CT synthesis
- CBCT-to-CT synthesis
- brain and pelvic anatomies
- multi-center variability
- preprocessing
- image registration
- anonymization
- train / validation / test design
- standardized benchmarking

The paper describes a multi-center dataset containing CT with registered MRI or CBCT data and emphasizes the need for comparable evaluation across synthetic-CT methods.

> **Note:** This reference is included for scientific and benchmarking context. Its inclusion here should not be interpreted as a statement that the private clinical cohort documented in this repository is the SynthRAD2023 dataset.

### Public Dataset Resources

**Training collection:**  
https://doi.org/10.5281/zenodo.7260704

**Validation collection:**  
https://doi.org/10.5281/zenodo.7868168

---

## 5. SynthRAD2023 Challenge Report

### Huijben et al. — SynthRAD2023 Challenge Report

Huijben EMC, Terpstra ML, Galapon AJ, et al.  
**Generating synthetic computed tomography for radiotherapy: SynthRAD2023 challenge report.**  
*Medical Image Analysis.* 2024;97:103276.

**DOI:**  
https://doi.org/10.1016/j.media.2024.103276

### Relevance

The challenge report provides a large-scale comparison of modern synthetic-CT generation approaches.

Important observations include:

- comparison of MRI-to-CT and CBCT-to-CT methods
- multi-center benchmarking
- image-similarity evaluation
- dose-based evaluation
- comparison of different neural-network families
- analysis of the relationship between image metrics and clinical dose accuracy

A particularly important methodological lesson is that strong image-similarity metrics alone do not necessarily guarantee equivalent dosimetric performance.

This supports the broader principle that synthetic-CT evaluation should extend beyond a single global image metric.

---

# Image-to-Image Translation Foundations

## 6. Pix2Pix

### Isola et al. — Conditional Adversarial Image Translation

Isola P, Zhu J-Y, Zhou T, Efros AA.  
**Image-to-Image Translation with Conditional Adversarial Networks.**  
*IEEE Conference on Computer Vision and Pattern Recognition (CVPR).* 2017:1125–1134.

**DOI:**  
https://doi.org/10.1109/CVPR.2017.632

### Relevance

Pix2Pix provides a foundational paired image-to-image translation framework based on conditional GANs.

Key ideas include:

- paired input / target images
- conditional adversarial learning
- generator / discriminator training
- U-Net-based generator
- PatchGAN discriminator
- combined adversarial and L1 objectives

The framework influenced later medical image-synthesis approaches, including MRI-to-CT translation.

---

## 7. CycleGAN

### Zhu et al. — Unpaired Image Translation

Zhu J-Y, Park T, Isola P, Efros AA.  
**Unpaired Image-to-Image Translation using Cycle-Consistent Adversarial Networks.**  
*IEEE International Conference on Computer Vision (ICCV).* 2017:2223–2232.

**DOI:**  
https://doi.org/10.1109/ICCV.2017.244

### Relevance

CycleGAN introduced a widely used framework for image translation when aligned input-target pairs are unavailable.

Core concepts include:

- unpaired learning
- adversarial translation
- forward and inverse mappings
- cycle-consistency loss

This work is relevant to the broader synthetic-CT literature because paired medical images can be difficult to acquire and perfectly register.

---

# Evaluation Context

## Image Similarity Metrics

The project used complementary image-quality measures including:

- **MAE — Mean Absolute Error**
- **SSIM — Structural Similarity Index**
- **PSNR — Peak Signal-to-Noise Ratio**

These metrics were interpreted alongside:

- tissue-level HU analysis
- anatomical coverage
- case-level failure inspection
- cross-cohort generalization

The evaluation approach intentionally avoided treating a single global metric as a complete description of model behavior.

---

# How These References Informed the Project

The literature contributed to several layers of the work:

| Area | Key References |
|---|---|
| MRI radiotherapy context | Schmidt & Payne |
| MRI-only / substitute CT | Edmund & Nyholm |
| Deep-learning sCT landscape | Spadea et al. |
| Modern dataset & benchmarking design | Thummerer et al. |
| Large-scale sCT evaluation | Huijben et al. |
| Paired image translation | Isola et al. |
| Unpaired image translation | Zhu et al. |

Together, these references helped frame the project around more than model architecture alone.

They reinforced the importance of:

- data quality
- registration
- paired versus unpaired learning
- reproducible evaluation
- anatomical analysis
- cross-cohort generalization
- clinically meaningful validation

---

## Related Documentation

- [Main Case Study](../README.md)
- [Experimental History](experiments.md)
- [Evaluation & Failure Analysis](evaluation.md)
- [Methodology](methodology.md)

---

## Citation Note

This repository is a technical portfolio case study rather than a formal publication.

References are provided to document the scientific context and foundational literature that informed the work.

The project-specific experimental results described elsewhere in this repository should not be interpreted as results reported by the cited papers unless explicitly stated.
