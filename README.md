# Earthquake Impact & Vulnerability Analysis  

## Abstract  
This project delivers a comprehensive analysis pipeline—from raw earthquake catalogs through felt‑shaking estimation, spatial buffering, building vulnerability scoring, and statistical modeling—providing interactive and static outputs for planners, insurers, and researchers.

---

## 1. Introduction & Motivation  
Earthquakes threaten built environments, yet epicenter plots alone do not convey felt intensity nor structural exposure. Our goals were to:

1. **Compute ground shaking metrics**: apply the Boore & Atkinson (2008) GMPE to estimate Peak Ground Acceleration (PGA) and convert to Modified Mercalli Intensity (MMI) via Atkinson & Kaka (2007) [2], referencing the USGS MMI scale definitions [1].  
2. **Visualize seismicity** interactively using Folium: individual events, density heatmaps, and KMeans clustering.  
3. **Generate felt‑intensity buffers**: buffer epicenters by MMI-based radii (interpolated or rounded) to map potential impact zones.  
4. **Overlay parcel data**: intersect buffers with Orange County property parcels to compute exposure counts.  
5. **Score vulnerability**: assign base‑year and improvement‑value risk scores, plus fault‑proximity intersections.  
6. **Model exposure-risk relationships**: use linear and polynomial regression to quantify how seismic exposure correlates with structural vulnerability.

This workflow replaces a static report with a fully reproducible code base and rich visual outputs.

---

## 2. Data Sources & Preprocessing  
| File                                    | Description                                                                                         |
|-----------------------------------------|-----------------------------------------------------------------------------------------------------|
| `data/USGS_query.csv`                   | Raw USGS earthquake catalog (time, lat, lon, depth, magnitude, place, etc.)                          |
| `data/Cali_faults.geojson`              | California fault‑line geometries                                                                    |
| `data/orange_county_parcel_clean2.csv`  | Orange County parcel records (coordinates, improvement value, base‑year)                            |

1. **Earthquake data**: downloaded from USGS, saved as CSV.  
2. **Parcel data**: filtered to valid coordinates and nonzero improvement values.  
3. **Fault lines**: projected to EPSG:4326 for consistency.

---

## 3. Methodology  

### 3.1 Ground Motion Prediction (PGA → MMI)  
- Implemented the Boore & Atkinson (2008) [3] GMPE in `scripts/compute_intensity.py`.  
  - **Inputs**: magnitude (`mag`), fixed site distance = 50 km.  
  - **Output**: `pga` in g, clamped to avoid numerical extremes.  
- Converted PGA to MMI using Atkinson & Kaka (2007) [2]:  
  ```math
  MMI = 3.10 \log_{10}(PGA) + 3.92

### 3.2 Interactive Web Mapping  
Scripts in `scripts/map_quakes.py` produce three HTML maps:  
- **Epicenter Map**: circle markers sized by magnitude, popups with `mag`.  
- **Heatmap**: kernel density weighted by magnitude.  
- **Clustering**: KMeans (n=5) on latitude/longitude, cluster centers sized by average magnitude.  

### 3.3 Spatial Buffers & Parcel Overlay  
Handled in `scripts/buffer_and_plot.py`:  
- **MMI‑based buffers**: fractional interpolation and integer rounding using a radius lookup.  
- **GeoPandas/Shapely**: buffer epicenters, overlay parcels, count intersections → exposure per parcel.  
- **Cartopy**: static California map with coastlines, borders, states, fault lines, quake buffers, and parcels colored by vulnerability.  

### 3.4 Vulnerability Scoring & Regression Modeling  
Implemented in `scripts/risk_modeling.py`:  
- **Parcel vulnerability scores**:  
  - **Base‑year score**: older buildings are more vulnerable.  
  - **Improvement‑value score**: lower parcel value = higher vulnerability.  
  - **Fault intersections**: extra exposure if parcel lies on a fault.  
  - **Structural risk** = Base‑year score + Improvement‑value score.  
- **Statistical analysis**:  
  - Linear regression (Normal Equation + scikit‑learn).  
  - Polynomial regression (degrees 0–17) to showcase under/overfitting.  
  - Model evaluation via Mean Squared Error on train/test splits.  

---

## 4. Results & Interpretation  
- **GMPE outputs**: PGA ranged 0.01–0.2 g → MMI III–VII.  
- **Folium maps**: epicenter clusters align with West Coast tectonics; heatmap highlights the San Andreas corridor.  
- **Buffer plots**: reveal potential impact zones and populated overlaps.  
- **Parcel scores**: exposure counts 0–12; structural risk scores 2–20, highest near faults in older, low‑value areas.  
- **Regression**: linear models achieved R² ≈ 0.5; polynomial fits above degree 5 overfit (test MSE ↑).  

---

## 5. Limitations & Future Work

Fixed distance assumption: using a constant 50 km can misestimate PGA for near-field events.
Lookup-based buffers: refine with ShakeMap or spectral gridding.
Geographic scope: extend parcel analyses beyond Orange County.
Scoring granularity: incorporate building codes, occupancy, and soil amplification.
Scenario modeling: add hazard simulations and temporal analyses.

## 6. References

1) USGS, Earthquake Hazards Program. The Modified Mercalli Intensity Scale. U.S. Geological Survey.
https://www.usgs.gov/programs/earthquake-hazards/modified-mercalli-intensity-scale
2) Atkinson, G. M. & Kaka, S. I. (2007). Relationships between Felt Intensity and Instrumental Ground Motion in the Central United States and California. Bulletin of the Seismological Society of America, 97(2): 497–510. https://doi.org/10.1785/0120060154
3) Boore, D. M. & Atkinson, G. M. (2008). Ground‑Motion Prediction Equations for the Average Horizontal Component of PGA, PGV, and 5%-Damped PSA at Spectral Periods between 0.01 and 10.0 s. Bulletin of the Seismological Society of America, 98(3): 1121–1138.
4) Wald, D. J., Quitoriano, V., Heaton, T. H. & Kanamori, H. (1999). Relationships between Peak Ground Acceleration, Peak Ground Velocity, and Modified Mercalli Intensity in California. Earthquake Spectra, 15(3): 557–584.
---
## 7. Directory Structure  
```bash
earthquake-analysis/
├── data/
│   ├── USGS_query.csv
│   ├── Cali_faults.geojson
│   └── orange_county_parcel_clean2.csv
├── outputs/
│   ├── earthquake_locations.html
│   ├── earthquake_heatmap.html
│   ├── earthquake_cluster_map.html
│   ├── Processed_Earthquake_Data.csv
│   ├── parcel_with_earthquake_scores.csv
│   └── earthquake_mmi_buffer_map.png
├── scripts/
│   ├── map_quakes.py
│   ├── compute_intensity.py
│   ├── buffer_and_plot.py
│   └── risk_modeling.py
├── requirements.txt
└── README.md

---
