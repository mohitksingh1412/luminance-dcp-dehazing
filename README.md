# Luminance-Based Transmission and Atmospheric Light Refinement for Single Image Dehazing

[![License: Non-Commercial Research]([suspicious link removed])](LICENSE)

> **Official implementation for Luminance-Based Transmission and Atmospheric Light Refinement for Single Image Dehazing"**  
> *Author: Dr. Mohit Kumar Singh, Department of Computer Science & Engineering, MNIT Jaipur*

---

## 📌 Overview

Traditional physics-based single-image dehazing techniques—most notably He et al.'s **Dark Channel Prior (DCP)**—rely on estimating the transmission map $t(x)$ and global atmospheric light $L_\infty$ using minimum filters. However, this assumption frequently leads to severe structural degradation, including:
1. **Halo and blocking artifacts** around sudden depth discontinuities and sharp boundaries.
2. **Color distortion and over-saturation** in high-luminance, low-contrast regions (e.g., sky regions, bright pavements, and white objects).

This repository presents a novel **Model Restoration Technique (MRT)** that operates within the **YCbCr color space** by decoupling the **luminance ($Y$) channel**, where haze concentrates. By generating multi-exposure underexposed representations, extracting edge features using guided filters, and fusing luminance and RGB transmission maps, this framework reconstructs crisp, natural-looking images with intact color fidelity.

---

## 🔬 Limitations of Dark Channel Prior (DCP)

<p align="center">
  <img src="assets/figures/dcp_limitations.png" alt="DCP Limitations" width="85%"/>
  <br>
  <em>Figure 1: Common failure modes of traditional DCP: (Left) Halo artifacts near sky transitions, (Center) Over-saturation and severe color distortion, and (Right) Blocking artifacts across depth edges.</em>
</p>

---

## ⚙️ Methodology & Architecture

The proposed dehazing framework follows an end-to-end multi-stage pipeline:

<p align="center">
  <img src="assets/figures/pipeline_flowchart.png" alt="Proposed Framework Architecture" width="95%"/>
  <br>
  <em>Figure 2: Complete architectural pipeline of the luminance-based transmission and atmospheric light refinement framework.</em>
</p>

### 1. Luminance Decomposition & Multi-Exposure Generation
Hazy images exhibit abnormally elevated luminance and suppressed contrast. The input hazy image $I_{in}$ is converted to the **YCbCr** color space to isolate the luminance component $Y$. To recover fine contrast without information loss, a bank of underexposed images is generated via power transformation using gamma coefficients $\gamma > 1$:
$$I_{GC}^k = C \cdot I_{in}^\gamma, \quad \gamma \in \{1.2, 2, 4, 8\}$$

<p align="center">
  <img src="assets/figures/gamma_exposures.png" alt="Gamma Exposure Adjustment" width="90%"/>
  <br>
  <em>Figure 3: Exposure adjustment using gamma correction on hazy and luminance components across varying gamma values ($\gamma = 1.2, 2, 4, 8$).</em>
</p>

### 2. Edge-Preserving Guided Filtering & Weighted Fusion
For each underexposed image $I_{GC}^k$, structural and textural boundary features are extracted using an edge-preserving **Guided Filter** $F_{r,b}$, with the image acting as its own guidance:
$$E_{map}^k = F_{r,b}(I_{GC}^k, I_{GC}^k), \quad k \in \{1, 2, 3, 4\}$$

Each edge map is assigned an exposedness weight $W_k$, and a high-fidelity structural representation is synthesized via weighted fusion:
$$FusedE_{map} = \sum_{k=1}^4 W_k \cdot E_{map}^k$$

<p align="center">
  <img src="assets/figures/edge_maps_and_fusion.png" alt="Guided Edge Maps and Fusion" width="90%"/>
  <br>
  <em>Figure 4: (a–d) Edge maps $E_{map}^k$ from guided filtering, (e–h) exposedness weight maps $W_k$, (i) final fused edge map $FusedE_{map}$, and (j) luminance-derived transmission map $\hat{t}_Y(x,y)$.</em>
</p>

### 3. Transmission Map Estimation & Refinement
* **Luminance Transmission:** Dark Channel Prior is applied to the fused edge map to preserve high-frequency details:
  $$\hat{t}_Y(x, y) = 1 - 0.95 \cdot I_{darkYnorm}(x, y)$$
* **RGB Channel Transmission:** Estimated across individual color channels:
  $$\hat{t}_{rgb}(x, y) = 1 - \frac{0.95}{3} \sum_{c \in \{r,g,b\}} I_{dark\,c\,norm}(x, y)$$
* **Transmission Refinement:** The refined transmission $\hat{t}_{ref}(x, y)$ is obtained by averaging $\hat{t}_Y$ and $\hat{t}_{rgb}$, effectively eliminating boundary halos while properly handling sky regions.

<p align="center">
  <img src="assets/figures/transmission_comparison.png" alt="Transmission Comparison" width="85%"/>
  <br>
  <em>Figure 5: Qualitative comparison of estimated transmission maps: Traditional DCP (middle) vs. Proposed Luminance-refined transmission map (right). Note the absence of halos near trees and building edges.</em>
</p>

### 4. Non-Uniform Atmospheric Light Estimation
To counter non-uniform illumination and nighttime conditions, atmospheric light $L_\infty$ is refined using **Channel-Difference (CD)** guided filtering:
$$I_{CD}(x, y) = \max_{c \in \{R,G,B\}}(I_c(x, y)) - \min_{c \in \{R,G,B\}}(I_c(x, y))$$
$$\hat{L}_\infty^c = GIF_{r,b}(I_{CC}(x, y), I_{CD}(x, y))$$

### 5. Scene Radiance Recovery
Using the inverse atmospheric scattering model, the dehazed image $R(x, y)$ is restored as:
$$R(x, y) = \frac{I_{in}(x, y) - \hat{L}_\infty}{\max(\hat{t}_{ref}(x, y), t_0)} + \hat{L}_\infty$$

---

## 📊 Experimental Results

### Qualitative Comparisons

<p align="center">
  <img src="assets/results/outdoor_ots_comparison.png" alt="Outdoor Visual Comparison" width="92%"/>
  <br>
  <em>Figure 6: Visual dehazing comparison on RESIDE-OTS outdoor images (e.g., Beijing Railway Station & Forest Street). Left to right: Hazy input, DCP, Proposed Method, Ground Truth.</em>
</p>

<p align="center">
  <img src="assets/results/real_world_comparison.png" alt="Real World Comparison" width="92%"/>
  <br>
  <em>Figure 7: Dehazing results on challenging real-world daytime images with large building complexes and sky areas.</em>
</p>

---

### Quantitative Evaluation

The proposed method was benchmarked extensively against state-of-the-art algorithms on the **RESIDE** benchmark dataset (**RESIDE-SOTS** indoor and **RESIDE-OTS** outdoor images):

#### 1. Outdoor Benchmark (OTS Synthetic Images)
| Image ID | DCP PSNR (dB) | Proposed PSNR (dB) | PSNR Gain (%) | DCP SSIM | Proposed SSIM | SSIM Gain (%) |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **0001** | 14.55 | **21.12** | **+45.10%** | 0.796 | **0.873** | **+9.63%** |
| **0004** | 11.82 | **15.68** | **+32.62%** | 0.754 | **0.812** | **+7.66%** |
| **0014** | 12.66 | **16.39** | **+29.47%** | 0.657 | **0.804** | **+22.39%** |
| **0042** | 11.39 | **21.18** | **+85.96%** | 0.695 | **0.812** | **+16.80%** |
| **0253** | 14.35 | **21.36** | **+48.89%** | 0.721 | **0.826** | **+14.48%** |
| **Average (OTS)** | **14.56** | **19.14** | **+31.45%** | **0.741** | **0.828** | **+11.74%** |

#### 2. Indoor Benchmark (SOTS Indoor Images)
| Image ID | DCP PSNR (dB) | Proposed PSNR (dB) | DCP SSIM | Proposed SSIM |
| :---: | :---: | :---: | :---: | :---: |
| **1407** | 14.21 | **18.73** (+31.8%) | 0.808 | **0.893** (+10.5%) |
| **1409** | 17.16 | **22.14** (+29.0%) | 0.831 | **0.906** (+9.1%) |
| **1417** | 15.96 | **21.79** (+36.6%) | 0.856 | **0.884** (+3.3%) |
| **1435** | 17.49 | **20.95** (+19.8%) | 0.807 | **0.843** (+4.5%) |

---

## 🚀 Quick Start

### 1. Prerequisites & Installation
pip install -r requirements.txt

2. Dependencies
numpy >= 1.20.0
opencv-python >= 4.5.0
scipy >= 1.7.0
matplotlib >= 3.4.0

3. Usage
To dehaze a single image:
python main.py --input data/sample_inputs/hazy_street.png --output data/sample_outputs/dehazed_street.png --gamma 1.2 2.0 4.0 8.0
To run batch evaluation over the SOTS / OTS test sets:
python evaluate.py --dataset_dir ./data/RESIDE_SOTS --save_dir ./results/

📖 Citation
If you find this research or code useful in your work, please cite the thesis:
@phdthesis{singh2025visibility,
  author    = {Mohit Kumar Singh},
  title     = {Visibility Enhancement for Hazy Environment},
  school    = {Malaviya National Institute of Technology (MNIT) Jaipur},
  year      = {2025},
  address   = {Jaipur, India},
  chapter   = {3: Luminance-based Transmission and Atmospheric light Refinement}
}
