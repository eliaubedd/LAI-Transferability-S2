
# Multispectral LAI Retrieval from Sentinel-2

**Temporal Transferability and Physical Consistency Analysis**

An independent MSc research project investigating how well a data-driven model trained on one Sentinel-2 acquisition can reproduce **SNAP-derived Leaf Area Index (LAI)** and generalize to acquisitions from different seasons and years.

The central research question is not only *how accurately can LAI be approximated within one scene?*, but also *how stable is the learned reflectance–LAI relationship under temporal distribution shifts?*

## Research objectives

- Approximate LAI estimates produced by the **SNAP Sentinel-2 Biophysical Processor** using a Random Forest regressor.
- Examine the limitations of using NDVI alone, particularly its saturation at high LAI.
- Evaluate within-scene generalization using a **spatial train–test split** rather than a random pixel split.
- Measure **cross-season** and **cross-year** transferability without retraining on the target acquisitions.
- Interpret the resulting errors in relation to vegetation structure, spectral response, and unmodeled acquisition geometry.

## Data and reference target

| Property | Description |
|---|---|
| Sensor | Sentinel-2 MSI |
| Product | Level-2A surface reflectance |
| Analysis resolution | 20 m |
| Region of interest (ROI) | 20 km × 20 km, selected from an approximately 110 km × 110 km scene |
| Training acquisition | July 2021 |
| Cross-season evaluation | October 2021 |
| Cross-year evaluation | August 2024 |
| Prediction target | SNAP Biophysical Processor LAI estimates |

### ROI selection

A vegetation-focused ROI was chosen by computing NDVI at 10 m, spatially averaging it to 200 m, and applying a sliding-window search for the 20 km × 20 km area with the highest mean NDVI. The **same geographic ROI** was used for all three acquisitions.

### Features

The model receives seven per-pixel inputs, harmonized to a 20 m grid:

```text
[B05, B06, B07, B8A, B11, B12, NDVI]
```

- **B05, B06, B07:** Red-edge bands, sensitive to vegetation characteristics.
- **B8A:** Narrow near-infrared band, sensitive to canopy structure and scattering.
- **B11, B12:** Shortwave infrared bands, providing information related to water content.
- **NDVI:** Widely used vegetation index, but less sensitive in dense-canopy/high-LAI regimes.

The reference LAI rasters were generated using the SNAP Biophysical Processor, whose neural-network retrieval is trained on radiative-transfer simulations. Negative reference LAI values were treated as invalid, and remaining values were restricted to the range **[0, 6]** to reduce retrieval artifacts and outliers.

> **Interpretation note:** SNAP-derived LAI is a model-generated reference, **not an independent field measurement**. The reported metrics quantify agreement with SNAP estimates, not independently established accuracy against in-situ LAI.

## Modeling and evaluation

### 1. Preliminary NDVI-only baseline

A second-degree polynomial regression was used to examine whether NDVI alone could reproduce LAI. It captured the overall trend but systematically underestimated parts of the medium- and high-LAI range, consistent with NDVI saturation.

### 2. Multispectral Random Forest

A **Random Forest regressor** was trained on July 2021 data using the six reflectance bands and NDVI. For within-scene evaluation, spatially separated training and test regions were used to reduce overly optimistic estimates caused by correlations between neighboring pixels.

The model-reported feature importances assigned approximately **85%** of the total importance to NDVI. Red-edge and SWIR bands nevertheless contributed complementary information, particularly in denser-canopy conditions.

### 3. Temporal transfer experiments

The Random Forest trained on **July 2021** was evaluated **without retraining** on:

- **October 2021 (cross-season):** A different phenological period within the same year.
- **August 2024 (cross-year):** A comparable seasonal period in a different year.

All three experiments used the same ROI and consistent preprocessing. Performance was evaluated with **R², RMSE, and signed bias** against SNAP-derived LAI.

## Quantitative results

| Evaluation scenario | Mean reference LAI | R² ↑ | RMSE ↓ | Bias | Pixels with LAI > 4 |
|---|---:|---:|---:|---:|---:|
| **Within-scene spatial split (July 2021)** | 2.84 | **0.997** | **0.047** | -0.006 | 9.8% |
| **Cross-season (October 2021)** | 1.58 | **0.826** | **0.254** | +0.196 | 0.2% |
| **Cross-year (August 2024)** | 2.62 | **0.860** | **0.443** | -0.217 | 11.3% |

*Values are reported in the accompanying project study. RMSE and bias are expressed in LAI units. Higher R² and lower RMSE indicate better agreement with the SNAP reference.*

### Reading the results

**Within-scene (July 2021).** The model closely reproduced SNAP-derived LAI in spatially held-out regions of the original acquisition (R² = 0.997; RMSE = 0.047). This demonstrates strong within-scene approximation, not field-validated biophysical accuracy.

**Cross-season (October 2021).** Agreement declined to R² = 0.826 and RMSE = 0.254, with a positive bias (+0.196). The October scene contained substantially less dense vegetation than the training acquisition, and the model tended to overestimate LAI under the shifted canopy conditions.

**Cross-year (August 2024).** R² remained relatively high (0.860), but RMSE increased to 0.443 and bias became negative (-0.217). Errors were particularly pronounced in high-LAI conditions, suggesting sensitivity to changes in canopy structure and the distribution of dense vegetation.

**Why both metrics matter:** Cross-year R² is higher than cross-season R², even though cross-year RMSE is worse. The reference LAI distributions differ across scenes; R² alone therefore does not provide a complete comparison of absolute error or transfer robustness.

## Scientific findings

1. **Strong within-scene fit does not imply temporal robustness.** A model can approximate a reference processor closely in one acquisition while exhibiting substantially larger errors on later acquisitions.
2. **Temporal shifts introduce systematic error.** The observed bias changes from positive in October 2021 to negative in August 2024, consistent with differences in vegetation conditions and LAI distributions.
3. **Dense canopies are challenging.** NDVI loses sensitivity at high LAI, and the cross-year analysis revealed particularly poor generalization in the high-LAI regime.
4. **Reflectance–LAI relationships are physically influenced.** Canopy structure, phenology, illumination, and viewing geometry affect observed reflectance; a purely statistical mapping does not explicitly enforce these relationships.

## Limitations

- **Modeled reference:** The ground truth used for training and testing is SNAP-derived LAI, not in-situ LAI.
- **Single training acquisition:** Training on July 2021 limits the range of seasonal and interannual conditions represented.
- **Geographic scope:** The evaluation uses one fixed 20 km × 20 km ROI; broader geographic transfer was not tested.
- **No explicit BRDF/geometry correction:** Solar and view geometry were not modeled as input variables or explicitly normalized for BRDF effects.
- **Not a physics-informed model:** Spectral inputs are physically motivated, but the Random Forest has no explicit radiative-transfer or physical constraints.

## Future work

- Train on multiple acquisition dates to improve robustness to temporal distribution shifts.
- Include acquisition geometry and assess BRDF-aware normalization.
- Compare against independent field measurements of LAI when available.
- Investigate hybrid radiative-transfer / machine-learning approaches or physically constrained models.
- Evaluate generalization across additional locations, vegetation types, and years.

## Tools

Python · rasterio · NumPy · scikit-learn · Matplotlib · ESA SNAP

This repository documents the study and its analyses. Sentinel-2 L2A acquisitions and corresponding SNAP LAI outputs are required to reproduce the numerical experiments; data availability and exact execution steps depend on the repository contents.

## Scientific reference

Weiss, M. & Baret, F. (2016). *S2ToolBox Level 2 Products: Algorithm Theoretical Basis Document (ATBD).* ESA STEP, Issue V1.1.

## Author

**Elia Leonardo Martin**  
MSc in Artificial Intelligence for Science and Technology  
Individual academic project — *Physical Sensors and Systems for Environmental Imaging*  
**Course grade: 30/30**
