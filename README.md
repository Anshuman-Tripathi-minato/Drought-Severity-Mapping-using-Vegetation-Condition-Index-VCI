# Drought-Severity-Mapping-using-Vegetation-Condition-Index-VCI-
Drought severity assessment using MODIS NDVI time-series, Vegetation Condition Index (VCI), Google Earth Engine, and Python.

# 🌱 Drought Severity Mapping using Vegetation Condition Index (VCI)

## Project Status

🚧 **Work in Progress**

This project has recently been started and is currently under development.

The objective is to assess drought severity by analyzing vegetation health using multi-temporal MODIS NDVI datasets and calculating the Vegetation Condition Index (VCI) in Google Earth Engine.

---

# Project Overview

Drought monitoring is essential for agricultural planning, climate resilience, and environmental management. This project uses long-term satellite observations to detect vegetation stress and classify drought severity across the study area.

The workflow combines Remote Sensing, GIS, and introductory Machine Learning concepts for spatial drought assessment.

---

# Objective

Develop an automated drought monitoring workflow using:

- Multi-temporal MODIS NDVI
- Vegetation Condition Index (VCI)
- Google Earth Engine
- Time-series satellite analysis
- Spatial drought classification
- Area statistics and thematic mapping

---

# Study Area

Bundelkhand Region (Uttar Pradesh – Madhya Pradesh, India)

*(Final Area of Interest will be updated during project development.)*

---

# Dataset

## Satellite Data

- MODIS NDVI Time-Series (2015–2023)

Optional

- Sentinel-2
- Landsat-8

---

# Ancillary Data

- Area of Interest (AOI)
- Administrative Boundary
- CHIRPS Rainfall Data (Validation)
- Google Earth Pro Reference Imagery

---

# Tools & Technologies

- Google Earth Engine
- Python
- JavaScript (GEE)
- QGIS
- GIS
- Remote Sensing

---

# Methodology

1. Define Area of Interest (AOI)
2. Load MODIS NDVI Time-Series
3. Generate Multi-temporal NDVI Dataset
4. Calculate NDVI Minimum
5. Calculate NDVI Maximum
6. Compute Vegetation Condition Index (VCI)
7. Classify Drought Severity
8. Apply Spatial Filtering
9. Convert Raster to Vector
10. Calculate Area Statistics
11. Prepare Final Drought Severity Map

---

# VCI Formula

VCI = (NDVI - NDVImin) / (NDVImax - NDVImin)

---

# Drought Classification

| VCI | Drought Level |
|------|---------------|
| < 0.20 | Extreme Drought |
| 0.20 – 0.40 | Moderate Drought |
| 0.40 – 0.60 | Mild Drought |
| > 0.60 | Healthy Vegetation |

---

# Expected Outputs

- Multi-temporal NDVI Maps
- VCI Raster
- Drought Severity Classification
- Vector Drought Zones
- Area Statistics
- Final Cartographic Map

---

# Repository Structure

```
drought-severity-vci/
│
├── data/
│   ├── raw/
│   ├── processed/
│   └── validation/
│
├── gee/
│   ├── load_data.js
│   ├── calculate_vci.js
│   ├── drought_classification.js
│
├── notebooks/
│
├── outputs/
│   ├── maps/
│   ├── figures/
│   └── reports/
│
├── docs/
│
├── README.md
└── requirements.txt
```

---

# Current Progress

- [x] Literature Review
- [x] Project Planning
- [ ] Define Study Area
- [ ] Load MODIS NDVI Dataset
- [ ] Generate NDVI Time-Series
- [ ] Calculate NDVI Min & Max
- [ ] Compute VCI
- [ ] Classify Drought Severity
- [ ] Generate Vector Layers
- [ ] Calculate Area Statistics
- [ ] Prepare Final Maps
- [ ] Documentation

---

# Future Improvements

- SPI and VCI Combined Analysis
- Integration with CHIRPS Rainfall Data
- Monthly Drought Monitoring Dashboard
- Time-Series Change Detection
- Machine Learning-based Drought Prediction
- Interactive Web GIS Visualization

---

# References

- Google Earth Engine
- MODIS NDVI Data Catalog
- QGIS Documentation
- CHIRPS Rainfall Dataset

---

## Author

Anshuman Tripathi

B.Tech Computer Science

Google Earth Engine • Remote Sensing • GIS • Python
