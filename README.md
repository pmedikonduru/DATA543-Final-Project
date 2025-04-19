Earthquake Impact & Vulnerability Analysis

An End-to-End, Reproducible Workflow

Abstract

This project delivers a comprehensive analysis pipeline—from raw earthquake catalogs through felt‑shaking estimation, spatial buffering, building vulnerability scoring, and statistical modeling—providing interactive and static outputs for planners, insurers, and researchers.

1. Introduction & Motivation

Earthquakes threaten built environments, yet epicenter plots alone do not convey felt intensity nor structural exposure. Our goals were to:

Compute ground shaking metrics: apply the Boore & Atkinson (2008) GMPE to estimate Peak Ground Acceleration (PGA) and convert to Modified Mercalli Intensity (MMI) via Atkinson & Kaka (2007) [2], referencing the USGS MMI scale definitions [1].

Visualize seismicity interactively using Folium: individual events, density heatmaps, and KMeans clustering.

Generate felt‑intensity buffers: buffer epicenters by MMI-based radii (interpolated or rounded) to map potential impact zones.

Overlay parcel data: intersect buffers with Orange County property parcels to compute exposure counts.

Score vulnerability: assign base‑year and improvement‑value risk scores, plus fault‑proximity intersections.

Model exposure-risk relationships: use linear and polynomial regression to quantify how seismic exposure correlates with structural vulnerability.

This workflow replaces a static report with a fully reproducible code base and rich visual outputs.

2. Data Sources & Preprocessing

File

Description

data/USGS_query.csv

Raw USGS earthquake catalog (time, lat, lon, depth, magnitude, place, etc.)

data/Cali_faults.geojson

California fault‑line geometries

data/orange_county_parcel_clean2.csv

Orange County parcel records (coordinates, improvement value, base‑year)

Earthquake data: downloaded from USGS, saved as CSV.

Parcel data: filtered to valid coordinates and nonzero improvement values.

Fault lines: projected to EPSG:4326 for consistency.

3. Methodology

3.1 Ground Motion Prediction (PGA → MMI)

Implemented the Boore & Atkinson (2008) [3] GMPE in compute_intensity.py.

Inputs: magnitude (mag), fixed site distance = 50 km.

Output: pga (g), clamped to avoid numerical extremes.

Converted PGAs to MMI using the empirical relation from Atkinson & Kaka (2007) [2]:

3.2 Interactive Web Mapping

Epicenter Map: circle markers sized by magnitude, popups showing mag.

Heatmap: kernel density weighted by magnitude.

Clustering: KMeans (n=5) on coordinates, cluster centers sized by average magnitude.

3.3 Spatial Buffers & Parcel Overlay

MMI Buffers: two approaches—fractional interpolation and integer rounding—using a lookup table for radii.

GeoPandas/Shapely: create point buffers, overlay parcels, and count intersections to derive an exposure count per parcel.

Cartopy: static California map with coastlines, states, borders, fault lines, quake buffers, and parcel points colored by vulnerability.

3.4 Vulnerability Scoring & Regression Modeling

Parcel scores:

Base‑year: older buildings score higher risk.

Improvement‑value: lower‑value buildings score higher risk.

Fault intersections: bonus if parcel lies on a fault.

Structural risk = Base‑year score + Improvement‑value score.

Statistical models:

Linear regression via Normal Equation and scikit‑learn.

Polynomial regression (degrees 0–17) to illustrate under/overfitting.

Evaluation by Mean Squared Error (MSE) on train/test splits.

4. Results & Interpretation

GMPE outputs: PGA ranged 0.01–0.2 g → MMI III–VII.

Folium maps: epicenters cluster along the West Coast; heatmap and clusters align with known tectonic features.

Buffers: illustrate MMI propagation and highlight populated overlap zones.

Parcel scores: exposure counts 0–12; structural risk scores 2–20, highest near faults in older, low‑value areas.

Regression: linear models (R² ≈ 0.5); polynomial fits above degree 5 overfit (test MSE ↑).

5. Directory Structure

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

6. Reproducibility & Usage

Setup environment

python3 -m venv venv  
source venv/bin/activate  
pip install -r requirements.txt  

Run scripts in sequence

python scripts/map_quakes.py  
python scripts/compute_intensity.py  
python scripts/buffer_and_plot.py  
python scripts/risk_modeling.py  

Explore outputs in the outputs/ folder and open HTML maps in a browser.

7. Limitations & Future Work

Fixed site distance may misestimate PGA for near‑field events; future work could use actual epicenter-to-site distances.

Lookup-based MMI buffers could be refined via ShakeMap or simulation grids.

Expand parcel datasets beyond Orange County for broader applicability.

Enhance vulnerability scoring with building code, occupancy type, and soil amplification.

Extend to scenario simulations, temporal analysis, or other hazards.

8. References

USGS, Earthquake Hazards Program. The Modified Mercalli Intensity Scale. U.S. Geological Survey.https://www.usgs.gov/programs/earthquake-hazards/modified-mercalli-intensity-scale

Atkinson, G. M. & Kaka, S. I. (2007). Relationships between Felt Intensity and Instrumental Ground Motion in the Central United States and California. Bulletin of the Seismological Society of America, 97(2): 497–510. https://doi.org/10.1785/0120060154

Boore, D. M. & Atkinson, G. M. (2008). Ground‑Motion Prediction Equations for the Average Horizontal Component of PGA, PGV, and 5%-Damped PSA at Spectral Periods between 0.01 and 10.0 s. Bulletin of the Seismological Society of America, 98(3): 1121–1138.

Wald, D. J., Quitoriano, V., Heaton, T. H. & Kanamori, H. (1999). Relationships between Peak Ground Acceleration, Peak Ground Velocity, and Modified Mercalli Intensity in California. Earthquake Spectra, 15(3): 557–584.
