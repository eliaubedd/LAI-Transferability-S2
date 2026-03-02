# 🌿 Multispectral LAI Retrieval from Sentinel-2

**Temporal Transferability and Physical Consistency Analysis**

## 📌 Overview

This project investigates the retrieval of **Leaf Area Index (LAI)** from **Sentinel-2 Level-2A surface reflectance** using a data-driven approach, and evaluates its temporal transferability across seasons and years.

The objective is twofold:

1. Reproduce SNAP Biophysical Processor LAI using a Random Forest model
2. Analyze the physical limitations and generalization capability of a purely data-driven approach

The study explicitly examines:

* Spectral sensitivity of red-edge and SWIR bands
* NDVI saturation at high LAI
* Spatial autocorrelation bias
* Cross-season and cross-year transferability
* Physical consistency between reflectance inputs and radiative-transfer-derived targets

---

## 🛰 Data

### Satellite Data

* **Sensor:** Sentinel-2 MSI
* **Processing Level:** L2A (Bottom-Of-Atmosphere Surface Reflectance)
* **Spatial Resolution:** 20 m (harmonized)

L2A reflectance includes:

* Atmospheric correction (Rayleigh scattering, aerosol correction, absorption)
* Solar irradiance normalization

Note: BRDF normalization is not explicitly applied.

---

## 🌱 Target Variable

* **LAI derived from SNAP Biophysical Processor**
* Based on radiative transfer modeling (PROSAIL-based inversion)

This introduces an important conceptual aspect:

> The target variable is physically constrained, while the model is purely data-driven.

---

## 🧠 Features

The input feature vector includes:

* Red-edge bands (B5, B6, B7)
* NIR band (B8A)
* SWIR bands (B11, B12)
* NDVI

### Physical Motivation

* **Red-edge:** Sensitive to chlorophyll content and canopy structural transitions
* **NIR:** Driven by internal leaf scattering and canopy multiple scattering
* **SWIR:** Sensitive to water absorption and vegetation moisture
* **NDVI:** Proxy for vegetation density (with known saturation limits)

All bands were resampled to 20 m for spatial consistency.

---

## 🌍 ROI Selection

Region of Interest (ROI) was selected using NDVI thresholding to isolate vegetated areas and reduce soil contamination.

---

## 🤖 Model

* **Algorithm:** Random Forest Regressor
* **Training strategy:** Spatial train-test split
* **Evaluation metrics:** R², RMSE

### Why Spatial Split?

To prevent artificial performance inflation due to spatial autocorrelation between neighboring pixels.

---

## 🔁 Temporal Transferability

Two transfer scenarios were tested:

* Cross-season transfer
* Cross-year transfer

The objective was to evaluate how well the learned reflectance–LAI relationship generalizes under:

* Changing solar zenith angle
* Seasonal canopy structural variations
* Phenological evolution
* Inter-annual environmental variability

---

## ⚠️ Known Physical Limitations

### NDVI Saturation

At high LAI, NDVI saturates due to:

* Red band absorption plateauing in upper canopy layers
* Continued NIR multiple scattering

### Geometry Effects

Reflectance depends on:

* Solar zenith angle
* Viewing geometry
* Canopy anisotropy (BRDF effects)

These were not explicitly modeled in the ML pipeline.

### Model Nature

The approach is:

* Data-driven
* Not physically constrained
* Not BRDF-normalized

This limits physical interpretability and long-term robustness.

---

## 🔬 Future Developments

### Hybrid RT–ML Approaches

Combine radiative transfer models and machine learning to improve physical consistency.

### Physically-Informed Modeling

Incorporate:

* Angular geometry
* Monotonic constraints
* Energy conservation principles
* RT-based simulation training

---

## 📊 Key Findings

* Random Forest can approximate SNAP-derived LAI with good performance under similar conditions.
* Temporal transferability degrades under strong seasonal or structural shifts.
* Purely data-driven models lack intrinsic physical robustness.

---

## 🧾 Project Structure

```
├── data/
├── preprocessing/
├── feature_engineering/
├── modeling/
├── evaluation/
└── figures/
```

---

## 🧪 Requirements

* Python 3.x
* rasterio
* numpy
* scikit-learn
* matplotlib

---

## 📖 Scientific Context

This work highlights the tension between:

* Physically-based radiative transfer modeling
* Purely statistical machine learning approaches

and explores the benefits and limitations of each within vegetation monitoring applications.

---

## 👨‍🎓 Author

Elia Leonardo Martin
Master's Degree Project – Remote Sensing
Grade: 30/30
