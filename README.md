# Wildfire Risk & Geospatial Analysis

**Python · GeoPandas · U.S. Census · MTBS wildfire perimeters · Machine learning evaluation**

## Project objective — what am I trying to find out?

**I am evaluating whether existing 2022 wildfire risk scores meaningfully distinguish U.S. properties exposed to mapped wildfires, and whether adding 2020 Census housing and demographic characteristics improves that evaluation.** The original dataset supplies a 1–10 risk score for each property, but **does not supply observed wildfire outcomes**. Rather than assume those scores are accurate, I construct an external exposure indicator by intersecting property coordinates with mapped 2022 wildfire perimeters, then examine how the scores perform.

This is an **evaluation and data-enrichment project**, not an attempt to recreate the original risk-scoring model or predict verified building damage.

## The problem I'm solving

A dataset of predicted risk scores alone cannot establish whether a model works. To evaluate it responsibly, I need an independent observation to compare against, a way to examine geographic differences, and a controlled test of whether additional variables help. Mapped wildfire perimeters provide an imperfect but testable exposure proxy; Census tract data provide an opportunity to investigate additional community-level context.

## My objectives and questions

1. **Assess the supplied risk scores.** Clean and profile the property data, inspect score distributions and geographic patterns, and determine whether higher scores are associated with mapped wildfire-perimeter intersection.
2. **Create an external evaluation target.** Spatially join property locations to 2022 MTBS wildfire perimeters, and compare score discrimination nationally and within more localized fire-affected areas.
3. **Enrich properties with Census information.** Assign each property to a 2020 Census tract, standardize tract GEOIDs, and create interpretable housing features such as vacancy rate and housing-type shares.
4. **Test whether enrichment adds predictive value.** Compare the original risk score alone, Census features alone, and both together using county-grouped cross-validation and ROC-AUC.
5. **Explain what the results do—and do not—show.** Identify geographic variation, limitations of the exposure proxy, and whether the added features improved performance in this experiment.

## What I actually do (end-to-end)

**Property records → quality checks and exploratory analysis → 2022 wildfire perimeter spatial intersection → exposure-based score evaluation → Census tract matching and feature engineering → geographic cross-validation → model comparison and interpretation.**

## Executive snapshot

| Dataset / result | Finding |
|---|---|
| Scope | 499,992 U.S. property records with pre-existing 2022 risk scores |
| Exposure proxy | 920 property points inside mapped 2022 wildfire perimeters (0.184%) |
| Overall risk-score ROC-AUC | Approximately 0.976 |
| Census tract matching | 499,965 properties matched (~99.99%) |
| Model comparison | Existing score alone performed best in the recorded county-grouped experiment |

**Main takeaway:** The supplied risk score strongly separates properties by the constructed perimeter-intersection target overall, but performance is weaker in some localized comparisons. Adding the selected Census features did not improve held-out county-grouped ROC-AUC in this experiment.

> **Data-use note:** The original property-level risk dataset and Census attribute extract were supplied for a technical assessment. They are **not redistributed** here. This portfolio documents the analysis and its methods; use of the underlying data requires appropriate authorization. The original risk model was not developed in this project.

## Why this matters

Wildfire risk estimates can inform screening, prioritization and further investigation. But an impressive aggregate score does not automatically answer whether the same model distinguishes two properties within the same fire-prone area. This project examines that distinction and tests whether tract-level housing information provides any additional signal. The analysis is exploratory and is not a deployable underwriting or emergency-response tool.

## Analytical questions

1. What do the supplied risk scores look like across properties and geographic areas?
2. Can mapped 2022 wildfire perimeters provide a useful, albeit imperfect, external outcome for evaluation?
3. How does model discrimination change when evaluated within fire-affected counties or nearby locations?
4. Can 2020 Census tract attributes be linked to properties, and do engineered housing features improve held-out geographic performance?

## Data sources

| Source | Role | Availability |
|---|---|---|
| Supplied property dataset | Coordinates, property identifiers and pre-existing 2022 risk scores | Restricted; not included |
| Supplied 2020 Census tract attributes | Housing and demographic enrichment | Restricted extract; not included |
| [U.S. Census Bureau 2020 cartographic boundaries](https://www.census.gov/geographies/mapping-files/2020/geo/carto-boundary-file.html) | Tract polygons and GEOID matching | Public source |
| [Monitoring Trends in Burn Severity (MTBS)](https://www.mtbs.gov/) | Mapped wildfire perimeters for 2022 | Public source |

The analysis uses a binary **perimeter-intersection proxy**: a property point is labeled positive if it intersects a mapped 2022 wildfire perimeter. **This does not mean the building was damaged**, and a negative intersection does not establish that the property was safe or undamaged.

## End-to-end analysis workflow

```text
Existing property risk scores + coordinates
              |
      Data quality and EDA
              |
2022 MTBS perimeter polygons --> Spatial intersection --> Exposure proxy
              |
     Overall + local validation
              |
2020 Census tract boundaries --> Point-in-polygon GEOID assignment
              |
 Census attribute merge + feature engineering
              |
County-grouped cross-validation and model comparison
              |
      Interpretation and limitations
```

## Technical workflow

### 1. Explore and validate the supplied risk scores

- Checked record counts, missing values, duplicate identifiers and score distributions.
- Analyzed risk-score patterns across geography and risk categories.
- Joined property points with 2022 MTBS perimeter polygons to create an external exposure proxy.
- Evaluated discrimination using ROC-AUC, precision–recall analysis and risk concentration measures.
- Compared overall performance with performance within fire-affected counties and approximately 25-km neighborhoods, to distinguish broad regional separation from more local ranking ability.

### 2. Enrich properties with Census data

**Data-integration challenge:** Census GEOIDs are identifiers, not quantities. The notebook identified **15,661** ten-character identifiers and standardized them to 11-character strings with leading-zero padding. It also reconciled property point coordinates in **EPSG:4326** with Census tract boundaries in **EPSG:4269** before the spatial match.


- Worked with point coordinates and Census tract polygons using GeoPandas and coordinate reference system alignment.
- Assigned tract GEOIDs using point-in-polygon spatial joins, then joined tract-level attributes.
- Engineered normalized housing features, including **vacancy rate**, **detached housing share**, and **mobile home share**.
- Examined how those tract characteristics varied across the existing risk-score categories.

### 3. Test incremental predictive value

- Compared three feature sets: **risk score only**, **Census features only**, and **risk score + Census features**.
- Used **five-fold county-grouped cross-validation** to reduce leakage from placing properties in the same county in both training and testing.
- Compared ROC-AUC across held-out groups and inspected permutation importance as a supplementary diagnostic.

## Results

### Wildfire exposure and geographic validation

| Measure | Notebook result |
|---|---:|
| Property locations evaluated | **499,992** |
| Points intersecting mapped 2022 wildfire perimeters | **920 (0.184%)** |
| Wildfire events represented by positive cases | **33** |
| Counties containing positive cases | **32** |
| Overall risk-score ROC-AUC against perimeter intersection | **~0.976** |
| ROC-AUC within fire-affected counties | **~0.951** |
| ROC-AUC within approximately 25-km neighborhoods | **~0.798** |
| Census tract matching rate | **~99.99%** |

The gap between the overall and more localized AUC values is an important finding: separating generally fire-prone regions is not the same as ranking properties accurately within nearby areas.

### Why evaluation scale changes the story

The overall ROC-AUC (~0.976) includes differences between geographically distant areas. The lower neighborhood-scale ROC-AUC (~0.798) suggests the task of distinguishing nearby properties is harder. These are different evaluation populations, not interchangeable estimates of property-level accuracy.

The project also examined perimeter-edge sensitivity: a 100-metre buffer changed the count of intersecting properties from **920 to 962**, with overall ROC-AUC moving from **0.9761 to 0.9744** in the recorded analysis. This is a useful robustness check, not proof that perimeter data are exact.

### County-grouped model comparison

| Feature set | Mean ROC-AUC | Standard deviation across folds |
|---|---:|---:|
| **Existing risk score only** | **0.9777** | **0.0092** |
| Census features only | 0.7187 | 0.2046 |
| Existing risk score + Census features | 0.7016 | 0.2331 |

**Interpretation:** In this particular county-grouped experiment, the existing score alone performed best. Adding the selected Census features did **not** improve held-out geographic discrimination. Large fold-to-fold variation in the Census-based models limits how broadly this result can be generalized. These comparisons use the constructed 2022 perimeter-intersection outcome, **not verified property damage**.

## Practical interpretation

For a decision-maker, the results suggest three priorities: **(1)** validate at the geographic scale where decisions are made, **(2)** avoid treating a perimeter-intersection label as verified property loss, and **(3)** require held-out evidence before accepting new features as improvements. The weaker combined-model result is a finding to investigate, not a reason to conclude that all Census data lack value.

## What this project demonstrates

- **Problem framing:** Found an external, measurable proxy when the supplied data contained no outcome labels.
- **Data engineering:** Integrated property coordinates, polygon boundaries and tract-level tabular attributes using spatial joins and GEOIDs.
- **Feature engineering:** Created interpretable ratios from raw housing counts for comparisons across tracts of different sizes.
- **Evaluation design:** Tested geographic generalization rather than relying only on a random property-level split.
- **Critical interpretation:** Reported both overall and localized discrimination, and distinguished wildfire exposure from structural loss.

## Visualizations

The executed notebook includes score distributions, geographic patterns, exposure-rate analyses, discrimination metrics and feature comparisons. The README intentionally does not embed property-level maps or records that have not been cleared for public redistribution. After reviewing the outputs, you can add approved, aggregated figures under an `images/` directory and link them here.

## Limitations and next steps

- **One year of observations:** A single wildfire season does not establish future predictive performance.
- **Imperfect outcome:** MTBS mapped perimeter intersection is not a verified property-loss label; unmapped fires and other exposure differences may affect the labels.
- **Spatial concentration:** Positive cases occur in a limited number of fires and counties, making some geographic folds unstable.
- **Coarse Census features:** Tract-level housing characteristics may miss parcel-scale differences in fuels, terrain, building materials or defensible space.
- **Ordinal risk score:** A 1–10 ranking should not be interpreted as a calibrated probability of damage.

Potential extensions include multiple fire seasons, verified property-damage records where available, and higher-resolution hazard features such as terrain, vegetation/fuel conditions and wildland–urban interface proximity.

## Methodological choices worth discussing in interviews

| Decision | Why it matters |
|---|---|
| Use MTBS perimeters | Creates a testable exposure proxy when the supplied dataset has predictions but no outcome labels |
| Preserve GEOIDs as strings | Avoids corrupting tract identifiers when leading zeros are present |
| Align coordinate reference systems | Prevents misleading spatial matches between differently referenced geometries |
| Normalize housing counts | Makes comparisons across tracts of different population or housing size more meaningful |
| Group folds by county | Reduces spatial leakage caused by nearby properties sharing geography and wildfire events |
| Report multiple geographic scales | Prevents an overall AUC from being mistaken for neighborhood-level performance |

## Tools and libraries

**Python**, pandas, NumPy, **GeoPandas**, Matplotlib, Plotly, SciPy, scikit-learn, spatial joins, CRS transformations, Census GEOIDs, ROC-AUC and grouped cross-validation.

## Repository structure

```text
wildfire-risk-geospatial-analysis/
├── README.md
├── requirements.txt
├── .gitignore
└── notebooks/
    └── wildfire_risk_analysis.ipynb
```

## Running the notebook

1. Use a supported Python environment with Jupyter or VS Code's notebook extension.
2. Install dependencies from the repository root:

   ```bash
   python -m pip install -r requirements.txt
   ```

3. Obtain authorized local copies of the three input files referenced by the notebook:
   - `wildfire_risk_data_set.xlsx`
   - `census_tract_level_data_2020.csv`
   - `cb_2020_us_tract_500k.zip`
4. Set `DATA_DIR` in the notebook's setup cell to the directory containing those files.
5. Restart the kernel and run the cells in order. Downloading MTBS perimeter data requires an internet connection.

**Reproducibility:** The full analysis cannot run from the public repository alone because the supplied assessment datasets are excluded. Saved notebook outputs illustrate the workflow but should be reviewed for redistribution restrictions before publishing.

## Further reading

- [MTBS — Monitoring Trends in Burn Severity](https://www.mtbs.gov/)
- [U.S. Census Bureau — 2020 Cartographic Boundary Files](https://www.census.gov/geographies/mapping-files/2020/geo/carto-boundary-file.html)
- [scikit-learn — StratifiedGroupKFold](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.StratifiedGroupKFold.html)

## Author

**Maira Taimoor**  
Data & Analytics Portfolio — Python, SQL, Geospatial Analysis, Machine Learning
