# 🌫️ Luminance-Based Transmission and Atmospheric Light Refinement for Single Image Dehazing

<p align="center">
  <img src="assets/figures/pipeline_flowchart.png" alt="Proposed Dehazing Framework" width="95%"/>
</p>

<h3 align="center">
  An Edge-Aware, Luminance-Guided Model Restoration Framework for Single Image Dehazing
</h3>

<p align="center">
  <b>Dr. Mohit Kumar Singh</b><br/>
  Department of Computer Science & Engineering<br/>
  Malaviya National Institute of Technology (MNIT), Jaipur, India
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Domain-Computer%20Vision-blue?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Research-Image%20Restoration-purple?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Method-Physics%20Based-success?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Status-Research%20Implementation-orange?style=for-the-badge"/>
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-Non--Commercial%20Research-lightgrey?style=flat-square"/></a>
  <img src="https://img.shields.io/badge/Dataset-RESIDE-blue?style=flat-square"/>
  <img src="https://img.shields.io/badge/Color%20Space-YCbCr-informational?style=flat-square"/>
</p>

---

## 📑 Table of Contents

- [Overview](#-overview)
- [Motivation and Research Contribution](#-motivation-and-research-contribution)
- [Limitations of Dark Channel Prior](#-limitations-of-dark-channel-prior-dcp)
- [Proposed Methodology](#-proposed-methodology)
  - [Luminance Decomposition and Multi-Exposure Generation](#1-luminance-decomposition--multi-exposure-generation)
  - [Edge-Preserving Guided Filtering and Weighted Fusion](#2-edge-preserving-guided-filtering--weighted-fusion)
  - [Transmission Estimation and Refinement](#3-transmission-map-estimation--refinement)
  - [Non-Uniform Atmospheric Light Estimation](#4-non-uniform-atmospheric-light-estimation)
  - [Scene Radiance Recovery](#5-scene-radiance-recovery)
- [Experimental Results](#-experimental-results)
- [Quantitative Evaluation](#-quantitative-evaluation)
- [Installation and Usage](#-installation-and-usage)
- [Project Structure](#-project-structure)
- [Citation](#-citation)
- [Acknowledgements](#-acknowledgements)

---

## 🔬 Overview

Single-image dehazing is a fundamental problem in computer vision, aiming to recover clear scene radiance from images degraded by atmospheric scattering.

Traditional physics-based approaches, particularly the **Dark Channel Prior (DCP)** proposed by He et al., estimate scene transmission and atmospheric light using local minimum operations. Although effective under many conditions, these techniques often introduce undesirable artifacts in complex scenes.

This repository presents a **Luminance-Based Transmission and Atmospheric Light Refinement Framework**, a model restoration technique operating in the **YCbCr color space**.

The proposed approach exploits luminance decomposition, multi-exposure underexposed representations, edge-preserving guided filtering, and weighted feature fusion to improve transmission estimation and atmospheric light recovery.

The framework aims to address:

- **Halo artifacts:** Structural degradation around sharp depth discontinuities and object boundaries.
- **Blocking artifacts:** Loss of local structural continuity caused by minimum-filter operations.
- **Color distortion:** Over-saturation and inaccurate color reconstruction in high-luminance regions.
- **Atmospheric light estimation:** Inaccurate global atmospheric light estimation under non-uniform illumination and nighttime conditions.

### ✨ Key Contributions

| Component | Contribution |
|---|---|
| Luminance decomposition | Separates luminance information using YCbCr color representation |
| Multi-exposure generation | Generates multiple underexposed representations using gamma transformations |
| Edge-aware feature extraction | Uses guided filtering to preserve structural and textural boundaries |
| Weighted feature fusion | Combines multi-exposure edge representations |
| Transmission refinement | Fuses luminance-derived and RGB-derived transmission maps |
| Atmospheric light refinement | Uses channel-difference guided filtering for improved atmospheric light estimation |
| Scene reconstruction | Recovers clear scene radiance using the atmospheric scattering model |

---

## 💡 Motivation and Research Contribution

The atmospheric scattering model describes the degradation of scene radiance due to atmospheric particles:

$$
I(x,y)=R(x,y)t(x,y)+L_\infty(x,y)(1-t(x,y))
$$

where:

- \(I(x,y)\): Observed hazy image.
- \(R(x,y)\): Original scene radiance.
- \(t(x,y)\): Transmission map.
- \(L_\infty(x,y)\): Atmospheric light.

The central challenge in single-image dehazing is accurately estimating the transmission map and atmospheric light from the observed hazy image.

The proposed framework introduces luminance-guided structural refinement to improve these estimations while preserving natural scene colors.

---

## ⚠️ Limitations of Dark Channel Prior (DCP)

<p align="center">
  <img src="assets/figures/dcp_limitations.png" alt="Limitations of Dark Channel Prior" width="90%"/>
</p>

<p align="center">
  <em>Figure 1. Common failure modes of traditional DCP: halo artifacts near sky transitions, color distortion and over-saturation, and blocking artifacts across depth discontinuities.</em>
</p>

Traditional DCP relies on the observation that most local image patches contain pixels with very low intensity in at least one RGB channel.

However, this assumption may fail in:

- Bright sky regions.
- White and light-colored objects.
- High-luminance surfaces.
- Regions with non-uniform illumination.

These limitations motivate the use of luminance-guided transmission estimation and edge-aware refinement.

---

## ⚙️ Proposed Methodology

The proposed framework consists of five major processing stages.

<p align="center">
  <img src="assets/figures/pipeline_flowchart.png" alt="Complete Proposed Framework Architecture" width="100%"/>
</p>

<p align="center">
  <em>Figure 2. Complete architectural pipeline of the proposed luminance-based transmission and atmospheric light refinement framework.</em>
</p>

### 1. Luminance Decomposition & Multi-Exposure Generation

Hazy images generally exhibit elevated luminance and reduced local contrast.

The input image \(I_{in}\) is converted into the **YCbCr color space** to isolate the luminance component \(Y\).

To recover fine structural information, multiple underexposed representations are generated using gamma power transformations.

The gamma-corrected representation is defined as:

$$
I_{GC}^{k}=C\cdot I_{in}^{\gamma_k}
$$

where:

$$
\gamma_k\in\{1.2,2,4,8\}
$$

and \(C\) represents the scaling constant.

The different exposure levels reveal complementary structural and textural information that may be suppressed in the original hazy image.

<p align="center">
  <img src="assets/figures/gamma_exposures.png" alt="Gamma Exposure Adjustment" width="95%"/>
</p>

<p align="center">
  <em>Figure 3. Multi-exposure representations generated through gamma correction on hazy images and luminance components.</em>
</p>

### 2. Edge-Preserving Guided Filtering & Weighted Fusion

For each underexposed representation \(I_{GC}^{k}\), an edge-preserving guided filter is applied to extract structural and textural boundary information.

The guided filtering operation is expressed as:

$$
E_{map}^{k}=F_{r,b}(I_{GC}^{k},I_{GC}^{k})
$$

where:

- \(F_{r,b}\): Guided filtering operator.
- \(r\): Local filtering radius.
- \(b\): Regularization parameter.
- \(k\): Exposure representation index.

Each extracted edge map is assigned an exposedness weight \(W_k\).

The weighted fusion operation is defined as:

$$
FusedE_{map}=\sum_{k=1}^{4}W_k\cdot E_{map}^{k}
$$

This fusion combines complementary structural information from multiple exposure levels into a unified representation.

<p align="center">
  <img src="assets/figures/edge_maps_and_fusion.png" alt="Guided Edge Maps and Weighted Fusion" width="95%"/>
</p>

<p align="center">
  <em>Figure 4. Guided edge maps, exposedness weight maps, fused edge representation, and luminance-derived transmission map.</em>
</p>

### 3. Transmission Map Estimation & Refinement

Transmission estimation is performed using two complementary representations: luminance-based transmission and RGB-based transmission.

#### A. Luminance Transmission

Dark Channel Prior is applied to the fused edge representation to preserve high-frequency structural information.

The luminance-derived transmission map is estimated as:

$$
\hat{t}_Y(x,y)=1-0.95\cdot I_{darkYnorm}(x,y)
$$

#### B. RGB Channel Transmission

The RGB transmission is estimated by considering normalized dark channel information across the three color channels:

$$
\hat{t}_{rgb}(x,y)=
1-\frac{0.95}{3}
\sum_{c\in\{r,g,b\}}
I_{dark\,c\,norm}(x,y)
$$

#### C. Transmission Refinement

The final refined transmission map is obtained by averaging the luminance-derived and RGB-derived transmission maps:

$$
\hat{t}_{ref}(x,y)=
\frac{\hat{t}_Y(x,y)+\hat{t}_{rgb}(x,y)}{2}
$$

This fusion combines luminance-based structural information with RGB-based atmospheric information, helping to reduce boundary halos and improve transmission estimation in challenging sky regions.

<p align="center">
  <img src="assets/figures/transmission_comparison.png" alt="Transmission Map Comparison" width="90%"/>
</p>

<p align="center">
  <em>Figure 5. Comparison between traditional DCP transmission estimation and the proposed luminance-refined transmission map. Improved structural continuity can be observed around trees and building boundaries.</em>
</p>

### 4. Non-Uniform Atmospheric Light Estimation

Accurate atmospheric light estimation is essential for recovering natural colors and maintaining scene consistency.

To address non-uniform illumination and challenging nighttime conditions, a channel-difference representation is generated.

The channel-difference image is defined as:

$$
I_{CD}(x,y)=
\max_{c\in\{R,G,B\}}I_c(x,y)
-
\min_{c\in\{R,G,B\}}I_c(x,y)
$$

The refined atmospheric light for each color channel is obtained using guided filtering:

$$
\hat{L}_\infty^c=
GIF_{r,b}(I_{CC}(x,y),I_{CD}(x,y))
$$

where:

- \(I_{CD}\): Channel-difference representation.
- \(I_{CC}\): Input guidance representation used by the atmospheric light refinement process.
- \(GIF_{r,b}\): Guided image filtering operator.
- \(\hat{L}_\infty^c\): Refined atmospheric light for channel \(c\).

This refinement is intended to better accommodate spatial variations in atmospheric illumination.

### 5. Scene Radiance Recovery

The final dehazed image is reconstructed using the inverse atmospheric scattering model.

The recovered scene radiance is expressed as:

$$
R(x,y)=
\frac{
I_{in}(x,y)-\hat{L}_\infty
}{
\max(\hat{t}_{ref}(x,y),t_0)
}
+\hat{L}_\infty
$$

where:

- \(I_{in}(x,y)\): Input hazy image.
- \(\hat{L}_\infty\): Refined atmospheric light.
- \(\hat{t}_{ref}(x,y)\): Refined transmission map.
- \(t_0\): Lower bound used to prevent numerical instability.

The reconstructed image represents the estimated haze-free scene radiance.

---

## 📊 Experimental Results

The proposed method was evaluated against traditional Dark Channel Prior on the **RESIDE benchmark dataset**, including:

- RESIDE-SOTS Indoor.
- RESIDE-OTS Outdoor.

The evaluation considers both perceptual image quality and structural similarity.

### Qualitative Evaluation

#### Outdoor Scenes – RESIDE-OTS

<p align="center">
  <img src="assets/results/outdoor_ots_comparison.png" alt="Outdoor Dehazing Results" width="95%"/>
</p>

<p align="center">
  <em>Figure 6. Qualitative comparison on RESIDE-OTS outdoor images, including challenging outdoor scenes. From left to right: Hazy Input, DCP, Proposed Method, and Ground Truth.</em>
</p>

#### Real-World Image Restoration

<p align="center">
  <img src="assets/results/real_world_comparison.png" alt="Real World Dehazing Comparison" width="95%"/>
</p>

<p align="center">
  <em>Figure 7. Visual dehazing results on challenging real-world daytime images containing large building complexes and sky regions.</em>
</p>

---

## 📈 Quantitative Evaluation

### 1. Outdoor Benchmark – RESIDE-OTS

The proposed method demonstrates improvements over traditional DCP across the reported test images.

| Image ID | DCP PSNR (dB) | Proposed PSNR (dB) | PSNR Gain (%) | DCP SSIM | Proposed SSIM | SSIM Gain (%) |
|:---:|---:|---:|---:|---:|---:|---:|
| **0001** | 14.55 | **21.12** | **+45.10%** | 0.796 | **0.873** | **+9.63%** |
| **0004** | 11.82 | **15.68** | **+32.62%** | 0.754 | **0.812** | **+7.66%** |
| **0014** | 12.66 | **16.39** | **+29.47%** | 0.657 | **0.804** | **+22.39%** |
| **0042** | 11.39 | **21.18** | **+85.96%** | 0.695 | **0.812** | **+16.80%** |
| **0253** | 14.35 | **21.36** | **+48.89%** | 0.721 | **0.826** | **+14.48%** |
| **Average (OTS)** | **14.56** | **19.14** | **+31.45%** | **0.741** | **0.828** | **+11.74%** |

### 2. Indoor Benchmark – RESIDE-SOTS

| Image ID | DCP PSNR (dB) | Proposed PSNR (dB) | DCP SSIM | Proposed SSIM |
|:---:|---:|---:|---:|---:|
| **1407** | 14.21 | **18.73 (+31.8%)** | 0.808 | **0.893 (+10.5%)** |
| **1409** | 17.16 | **22.14 (+29.0%)** | 0.831 | **0.906 (+9.1%)** |
| **1417** | 15.96 | **21.79 (+36.6%)** | 0.856 | **0.884 (+3.3%)** |
| **1435** | 17.49 | **20.95 (+19.8%)** | 0.807 | **0.843 (+4.5%)** |

> **Experimental observation:** The reported results indicate improved PSNR and SSIM compared with traditional DCP on the evaluated indoor and outdoor samples.

---

## 🚀 Installation and Usage

### 1. Prerequisites

- Python 3.8+
- OpenCV
- NumPy
- SciPy
- Matplotlib

### 2. Clone the Repository

```bash
git clone YOUR_REPOSITORY_URL
cd YOUR_REPOSITORY_NAME
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

Alternatively, install the dependencies individually:

```bash
pip install numpy>=1.20.0
pip install opencv-python>=4.5.0
pip install scipy>=1.7.0
pip install matplotlib>=3.4.0
```

### 4. Single Image Dehazing

Run the proposed framework on an individual hazy image:

```bash
python main.py \
    --input data/sample_inputs/hazy_street.png \
    --output data/sample_outputs/dehazed_street.png \
    --gamma 1.2 2.0 4.0 8.0
```

### 5. Batch Evaluation

Evaluate the proposed framework over the RESIDE SOTS/OTS dataset:

```bash
python evaluate.py \
    --dataset_dir ./data/RESIDE_SOTS \
    --save_dir ./results/
```

---

## 📁 Project Structure

```text
.
├── assets/
│   ├── figures/
│   │   ├── dcp_limitations.png
│   │   ├── pipeline_flowchart.png
│   │   ├── gamma_exposures.png
│   │   ├── edge_maps_and_fusion.png
│   │   └── transmission_comparison.png
│   │
│   └── results/
│       ├── outdoor_ots_comparison.png
│       └── real_world_comparison.png
│
├── data/
│   ├── sample_inputs/
│   └── sample_outputs/
│
├── main.py
├── evaluate.py
├── requirements.txt
├── LICENSE
└── README.md
```

---

## 📖 Citation

If you find this research or implementation useful in your academic work, please cite the following thesis:

```bibtex
@phdthesis{singh2025visibility,
  author    = {Mohit Kumar Singh},
  title     = {Visibility Enhancement for Hazy Environment},
  school    = {Malaviya National Institute of Technology (MNIT) Jaipur},
  year      = {2025},
  address   = {Jaipur, India},
  chapter   = {3: Luminance-based Transmission and Atmospheric light Refinement}
}
```

---

## 📜 License

This repository is intended for **non-commercial academic and research purposes**.

Please refer to the [LICENSE](LICENSE) file for the complete terms and conditions.

---

## 🙏 Acknowledgements

The author acknowledges the contributions of the computer vision research community and the RESIDE benchmark dataset, which provides a valuable foundation for evaluating single-image dehazing algorithms.

---

## 👨‍🔬 About the Author

**Dr. Mohit Kumar Singh**  
Ph.D. in Computer Engineering  
Department of Computer Science & Engineering  
Malaviya National Institute of Technology (MNIT), Jaipur, India

Research Interests:

`Computer Vision` · `Image Restoration` · `Image Dehazing` · `Deep Learning` · `Pattern Recognition` · `Generative AI`


<p align="center">
  <b>Advancing Computer Vision through intelligent image restoration and research-driven innovation.</b>
</p>
