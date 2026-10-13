# Wildfire Ignition Risk Map for Lesvos Island

A spatial logistic regression model that estimates relative wildfire ignition risk across Lesvos, Greece, from terrain and human-infrastructure variables. Built with ArcGIS Pro, Python (`arcpy`), and SPSS.

![Study area: Lesvos Island](AREAMAP.jpg)

## Key results

| Metric | Value |
|---|---|
| Sample Size | 795 points (397 fire occurrences, 398 unburned pseudo-absences) |
| Overall Classification Accuracy | 65.4% (520 of 795 points) |
| Sensitivity / Specificity | 68.3% / 62.6% |
| Hosmer-Lemeshow Test | χ² = 4.330, df = 8, p = 0.826 |
| Significant Predictors (p < 0.05) | 5 of 10 variables |
| Coordinate Reference System | Greek Grid / GGRS87 (EPSG:2100) |

The model demonstrates **moderate** predictive power and is designed as a relative susceptibility surface to support fire prevention planning, rather than a dynamic real-time fire forecast.

## Study area

Lesvos, situated in the northeastern Aegean Sea, is characterized by rugged topographies, a typical semi-arid Mediterranean climate, and a contiguous mosaic of pine forests, sclerophyllous scrubland, and agricultural olive groves. The island experiences recurring large-scale wildfires, such as the July 2022 fire that rapidly consumed mixed pine and agricultural zones, resulting in the evacuation of over 450 residents.

## Data

- **Fire Points:** Historic wildfire ignition points (Value: 1) sourced from official Hellenic Fire Service records.
- **No-Fire Points:** Random unburned points (Value: 0) generated across the study extent as a balanced pseudo-absence baseline.
- **Topography:** Digital Elevation Model (DEM) and derived slope gradients.
- **Human Infrastructure:** Euclidean distance surfaces calculated from settlements, agricultural parcels, road networks, power lines, and municipal landfills.

## Methodology

1. **Pre-processing (ArcGIS Pro):** Set analysis extent, cell size, and analysis mask matching the DEM. Generated continuous X and Y coordinate raster surfaces using Map Algebra in ArcGIS Pro (`arcpy.sa`). Derived slope gradients from the DEM. Stratified the road network into primary and secondary roads, computing separate Euclidean distance rasters.
2. **Normalization:** Rescaled every raster surface to a standardized 0–1 scale using min-max normalization in the Raster Calculator:
   $$X' = \frac{X - X_{\min}}{X_{\max} - X_{\min}}$$
3. **Sampling:** Applied *Extract Multi Values to Points* to sample all 10 normalized environmental predictors to the point database, assigning the binary `fire` indicator as the dependent variable.
4. **Statistical Modeling (SPSS):** Exported point attributes and fitted a Binary Logistic Regression with `fire` as the dependent response and the 10 normalized variables as independent predictors.
5. **Evaluation:** Evaluated model performance via classification tables (0.5 probability cut-off), sensitivity, specificity, and Hosmer-Lemeshow goodness-of-fit testing.
6. **Susceptibility Mapping:** Reconstructed the logistic function within the ArcGIS Pro Raster Calculator:
   $$P = \frac{1}{1 + e^{-z}}$$
   The resulting continuous probability surface was classified into 4 relative risk tiers (Low, Moderate, High, and Very High) using Natural Breaks (Jenks).

## Results

### Model Performance

| Class | Predicted No Fire | Predicted Fire | Classification Accuracy |
|---|---|---|---|
| **Observed No Fire** | 249 | 149 | 62.6% (Specificity) |
| **Observed Fire** | 126 | 271 | 68.3% (Sensitivity) |

*Overall Accuracy: 65.4% (Cut-off value: 0.5).* The overlapping distributions around the 0.5 threshold demonstrate moderate separation capacity.

### Significant Variables

| Predictor | Coefficient (B) | Significance (p) | Spatial Influence on Ignition Risk |
|---|---|---|---|
| **Y Coordinate** | 1.800 | < 0.001 | Increases towards the northern sector |
| **Distance to Landfills** | 2.390 | < 0.001 | Increases with distance from landfills |
| **Distance to Power Lines** | −2.743 | 0.002 | Increases in proximity to power lines |
| **Distance to Settlements** | −2.266 | 0.002 | Increases in proximity to wildland-urban interfaces |
| **Distance to Secondary Roads** | 0.898 | 0.036 | Increases with distance from secondary roads |

*Not statistically significant (p > 0.05):* Distance to main roads, distance to agricultural parcels, slope gradient, elevation, and X coordinate. In this landscape, human accessibility and infrastructure proved more predictive than topographic constraints.

### Risk Map

![Wildfire risk zone map of Lesvos](fire_risk_zone_map.jpg)

Very High and High risk zones are concentrated across the northern regions (Mithymna and Mantamados) and the eastern sector surrounding the capital Mytilene. Low risk zones dominate the agricultural lowlands around the Gulf of Kalloni and the arid western expanse near Eresos.

## Limitations

- **Omission of Dynamic Variables:** The model accounts exclusively for static spatial and infrastructure variables. Real-time meteorological factors (wind vectors, ambient humidity, temperature) and fuel moisture dynamics were not integrated.
- **Validation Scope:** Accuracy metrics were evaluated on the fitting dataset without an independent spatial cross-validation or out-of-sample test partition.
- **Geographic Surrogates:** The significance of the Y-coordinate acts as a spatial proxy rather than a direct causative mechanism, likely absorbing unmeasured microclimatic or fuel-loading trends across the north.
- **Future Improvements:** Implementation of multi-temporal Sentinel-2 vegetation indices (NDVI, NBR), spatial k-fold cross-validation, and non-linear ensemble algorithms (e.g., Random Forest, XGBoost).

## Tools & Environments

- **GIS Platform:** ArcGIS Pro (Spatial Analyst, Euclidean Distance, Slope, Map Algebra, Extract Multi Values to Points)
- **Scripting:** Python (`arcpy`)
- **Statistical Software:** IBM SPSS Statistics (Binary Logistic Regression)

## References

1. Cho, M., & Park, C. (2026). Predicting Anthropogenic Wildfire Occurrence Using Explainable Machine Learning Models: A Nationwide Case Study of South Korea. *Fire*, 9(3). https://doi.org/10.3390/fire9030126
2. Cobo-Muelas, E. S., López-Serrano, P. M., Wehenkel, C., Manzo-Delgado, L. de L., & Martínez-López, J. (2026). Wildfire Detection in the Iztaccíhuatl-Popocatépetl Protected Natural Area Using Spectral Indices and Logistic Regression. *Fire*, 9(2). https://doi.org/10.3390/fire9020050
3. Dimarco, N. A., Faraji, I., Wahbi, M., Maatouk, M., Boulaassal, H., Aalaoui, O. Y., & el Kharki, O. (2025). FireRisk-Zone-LR: A Logistic Regression-Based Wildfire Hazard Zoning Framework for Mediterranean Forests in Tangier, Morocco. *E3S Web of Conferences*, 676, 02006. https://doi.org/10.1051/e3sconf/202567602006
4. Karger, E., Jeppe, A., Ziolkowski, R., Korn, F., Hackel, T., Harr, M., Brée, T., & Ahlemann, F. (2026). Artificial intelligence for wildfire detection and management. *Discover Artificial Intelligence*, 6(1). https://doi.org/10.1007/s44163-026-01087-5
5. Liao, L., & Zhu, X. (2025). Predicting Wildfire Risk in Southwestern Saudi Arabia Using Machine Learning and Geospatial Analysis. *Remote Sensing*, 17(21). https://doi.org/10.3390/rs17213516
6. Marathianou, M., Kosmas, C., Gerontidis, S., & Detsis, V. (2000). Land-use evolution and degradation in Lesvos (Greece): A historical approach. *Land Degradation and Development*, 11(1), 63–73. https://doi.org/10.1002/(SICI)1099-145X(200001/02)11:1<63::AID-LDR369>3.0.CO;2-8
7. Vilar del Hoyo, L., Isabel, M. P. M., & Vega, F. J. M. (2011). Logistic regression models for human-caused wildfire risk estimation: Analysing the effect of the spatial accuracy in fire occurrence data. *European Journal of Forest Research*, 130(6), 983–996. https://doi.org/10.1007/s10342-011-0488-2
8. Copernicus (2022). Wildfire crisis: Lesvos, Greece. https://www.copernicus.eu/en/media/image-day-gallery/wildfire-crisis-europe-2022-lesvos-greece

---
*Project developed at the University of the Aegean, Department of Geography.*
