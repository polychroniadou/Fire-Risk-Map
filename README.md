# Wildfire Ignition Risk Map for Lesvos Island

A spatial logistic regression model that estimates relative wildfire ignition risk across Lesvos, Greece, from terrain and human-infrastructure variables. Built with ArcGIS Pro, Python and SPSS.

![Study area: Lesvos Island](AREAMAP.jpg)



## Key results

| | |
|---|---|
| Sample | 795 points (397 fire, 398 no-fire) |
| Overall classification accuracy | 65.4% (520 of 795 points) |
| Sensitivity / Specificity | 68.3% / 62.6% |
| Hosmer-Lemeshow test | χ² = 4.330, df = 8, p = 0.826 |
| Significant predictors (p < 0.05) | 5 of 10 variables |

The model has **moderate** predictive power. It is best read as a relative susceptibility map, not an exact probability forecast.

## Study area

Lesvos, in the northeastern Aegean Sea, has rugged terrain, a Mediterranean climate and a mix of semi-natural forest, shrubland and farmland. The island has a history of large wildfires, including the July 2022 fire that spread quickly through pine forest, scrub and cultivated land and led to the evacuation of 450 residents.

## Data

- **Fire points:** Recorded wildfire ignition points (value 1) sourced from official Hellenic Fire Service historic records.

- **No-fire points:** Random unburned points (value 0) generated across the study extent as a balanced pseudo-absence baseline.

- **Terrain:** Digital Elevation Model (DEM) and derived slope gradients.

- **Infrastructure:** Euclidian distance layers computed from settlements, agricultural parcels, road networks, power lines, and landfills.


## Methodology

1. **Pre-processing (ArcGIS Pro):** set the processing extent and mask to the DEM. Created X and Y coordinate rasters with Python. Generated continuous X and Y coordinate raster surfaces using Map Algebra in ArcGIS Pro (arcpy.sa).. Computed slope from the DEM. Split the road network into main (ID 1) and secondary (ID 2) roads and computed Euclidean distance rasters.
2. **Normalisation:** rescaled every raster to 0-1 with min-max normalisation in the Raster Calculator: `X' = (X − Xmin) / (Xmax − Xmin)`.
3. **Sampling:** used *Extract Multi Values to Points* to attach the raster values to each point, and created a binary `fire` field as the dependent variable.
4. **Model (SPSS):** exported the attribute table to CSV and ran a Binary Logistic Regression with `fire` as the dependent variable and the 10 normalised variables as predictors.
5. **Evaluation:** classification table (cut value 0.5), sensitivity and specificity, Hosmer-Lemeshow test and the observed groups and predicted probabilities plot.
6. **Risk map:** applied the fitted equation in the ArcGIS Raster Calculator, `P = 1 / (1 + e^(−z))`, and divided the result into 4 risk zones (low to very high) `divided the continuous probability surface into 4 relative risk zones (Low, Moderate, High, Very High) using Natural Breaks (Jenks)." (or Quantiles, whichever you used in ArcGIS)`.

## Results

### Model performance

| | Predicted no fire | Predicted fire |
|---|---|---|
| **Observed no fire** | 249 | 149 |
| **Observed fire** | 126 | 271 |

Sensitivity is 271 / (271 + 126) = 68.3% and specificity is 249 / (249 + 149) = 62.6%. The two classes overlap around the 0.5 cut value, so the model cannot separate them cleanly.

### Significant variables

| Variable | B | p | Effect on ignition risk |
|---|---|---|---|
| Y coordinate | 1.800 | < 0.001 | Higher in the north |
| Distance to landfills | 2.390 | < 0.001 | Increases with distance |
| Distance to power lines | −2.743 | 0.002 | Increases closer to power lines |
| Distance to settlements | −2.266 | 0.002 | Increases closer to settlements |
| Distance to secondary road | 0.898 | 0.036 | Increases with distance |

Not significant: distance to main road, distance to agricultural areas, slope, elevation and X coordinate. In this study area, human-infrastructure variables were more informative than topography.

### Risk map

![Wildfire risk zone map of Lesvos](fire_risk_zone_map.jpg)

Very high and high risk zones appear mainly in the north (around Mithymna and Mantamados) and in the east, including the area around Mytilene. Smaller high-risk patches are scattered across the island. The lowest risk is in the lowland plains, around the Gulf of Kalloni and in the west near Eresos.


## Limitations

- The model uses spatial variables only. Wind, humidity, temperature and fuel moisture are not included, and they strongly affect ignition and spread.
- Accuracy is measured on the same sample used to fit the model. There is no independent test set.
- No-fire points are random, and the sample is balanced 50/50 by design. The output is therefore a relative risk index, not the true probability of a fire at a location.
- The Y coordinate is significant but is a geographic proxy, not a cause. It likely captures effects the model doesn't measure directly.
- Significant distance variables show association, not causation. Distance to landfills and secondary roads may partly reflect how remote or forested an area is.
- Next steps: add climate and vegetation data (e.g. NDVI from Sentinel-2), validate on a held-out set or with spatial cross-validation, and compare with other models such as random forest.

## Tools

ArcGIS Pro (Raster Calculator, Euclidean Distance, Slope, Extract Multi Values to Points), Python (`arcpy`) and SPSS (Binary Logistic Regression).

## Reference

Cho, M., & Park, C. (2026). Predicting Anthropogenic Wildfire Occurrence Using Explainable Machine Learning Models: A Nationwide Case Study of South Korea. Fire, 9(3). https://doi.org/10.3390/fire9030126

Cobo-Muelas, E. S., López-Serrano, P. M., Wehenkel, C., Manzo-Delgado, L. de L., & Martínez-López, J. (2026). Wildfire Detection in the Iztaccíhuatl-Popocatépetl Protected Natural Area Using Spectral Indices and Logistic Regression. Fire, 9(2). https://doi.org/10.3390/fire9020050 

Dimarco, N. A., Faraji, I., Wahbi, M., Maatouk, M., Boulaassal, H., Aalaoui, O. Y., & el Kharki, O. (2025). FireRisk-Zone-LR: A Logistic Regression-Based Wildfire Hazard Zoning Framework for Mediterranean Forests in Tangier, Morocco. E3S Web of Conferences, 676, 02006. https://doi.org/10.1051/e3sconf/202567602006

Karger, E., Jeppe, A., Ziolkowski, R., Korn, F., Hackel, T., Harr, M., Brée, T., & Ahlemann, F. (2026). Artificial intelligence for wildfire detection and management. In Discover Artificial Intelligence (Vol. 6, Issue 1). Springer Nature. https://doi.org/10.1007/s44163-026-01087-5

Liao, L., & Zhu, X. (2025). Predicting Wildfire Risk in Southwestern Saudi Arabia Using Machine Learning and Geospatial Analysis. Remote Sensing, 17(21). https://doi.org/10.3390/rs17213516

Marathianou, M., Kosmas, C., Gerontidis, S., & Detsis, V. (2000). Land-use evolution and degradation in Lesvos (Greece): A historical approach. Land Degradation and Development, 11(1), 63–73. https://doi.org/10.1002/(SICI)1099-145X(200001/02)11:1<63::AID-LDR369>3.0.CO;2-8  

Vilar del Hoyo, L., Isabel, M. P. M., & Vega, F. J. M. (2011). Logistic regression models for human-caused wildfire risk estimation: Analysing the effect of the spatial accuracy in fire occurrence data. European Journal of Forest Research, 130(6), 983–996. https://doi.org/10.1007/s10342-011-0488-2 

Copernicus (2022). Wildfire crisis: Lesvos, Greece. From 
https://www.copernicus.eu/en/media/image-day-gallery/wildfire-crisis-europe-2022-lesvos-greece 

*Project developed at the University of the Aegean, Department of Geography.*
