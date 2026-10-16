# Wildfire Risk & Geospatial Analysis

**Python · GeoPandas · U.S. Census · MTBS wildfire perimeters · Machine learning evaluation**

## Project overview

How useful is an existing wildfire risk score when a dataset contains predictions but no observed outcomes? This project investigates that question using **499,992 U.S. property locations** with pre-existing 2022 wildfire risk scores on a **1–10 ordinal scale**. It combines exploratory analysis, external wildfire perimeter mapping, Census tract enrichment, feature engineering, and geographically grouped model evaluation.

The goal is not to build the original wildfire risk score. It is to **evaluate that supplied score**, identify where geographic effects matter, and test whether Census-derived housing characteristics add predictive value.

> **Data-use note:** The original property-level risk dataset and Census attribute extract were supplied for a technical assessment. They are **not redistributed** here. This portfolio documents the analysis and its methods; use of the underlying data requires appropriate authorization. The original risk model was not developed in this project.

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

## Technical workflow

### 1. Explore and validate the supplied risk scores

- Checked record counts, missing values, duplicate identifiers and score distributions.
- Analyzed risk-score patterns across geography and risk categories.
- Joined property points with 2022 MTBS perimeter polygons to create an external exposure proxy.
- Evaluated discrimination using ROC-AUC, precision–recall analysis and risk concentration measures.
- Compared overall performance with performance within fire-affected counties and approximately 25-km neighborhoods, to distinguish broad regional separation from more local ranking ability.

### 2. Enrich properties with Census data

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

### County-grouped model comparison

| Feature set | Mean ROC-AUC | Standard deviation across folds |
|---|---:|---:|
| **Existing risk score only** | **0.9777** | **0.0092** |
| Census features only | 0.7187 | 0.2046 |
| Existing risk score + Census features | 0.7016 | 0.2331 |

**Interpretation:** In this particular county-grouped experiment, the existing score alone performed best. Adding the selected Census features did **not** improve held-out geographic discrimination. Large fold-to-fold variation in the Census-based models limits how broadly this result can be generalized. These comparisons use the constructed 2022 perimeter-intersection outcome, **not verified property damage**.

## What this project demonstrates

- **Problem framing:** Found an external, measurable proxy when the supplied data contained no outcome labels.
- **Data engineering:** Integrated property coordinates, polygon boundaries and tract-level tabular attributes using spatial joins and GEOIDs.
- **Feature engineering:** Created interpretable ratios from raw housing counts for comparisons across tracts of different sizes.
- **Evaluation design:** Tested geographic generalization rather than relying only on a random property-level split.
- **Critical interpretation:** Reported both overall and localized discrimination, and distinguished wildfire exposure from structural loss.

## Limitations and next steps

- **One year of observations:** A single wildfire season does not establish future predictive performance.
- **Imperfect outcome:** MTBS mapped perimeter intersection is not a verified property-loss label; unmapped fires and other exposure differences may affect the labels.
- **Spatial concentration:** Positive cases occur in a limited number of fires and counties, making some geographic folds unstable.
- **Coarse Census features:** Tract-level housing characteristics may miss parcel-scale differences in fuels, terrain, building materials or defensible space.
- **Ordinal risk score:** A 1–10 ranking should not be interpreted as a calibrated probability of damage.

Potential extensions include multiple fire seasons, verified property-damage records where available, and higher-resolution hazard features such as terrain, vegetation/fuel conditions and wildland–urban interface proximity.

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

## Author

**Maira Taimoor**  
Data & Analytics Portfolio — Python, SQL, Geospatial Analysis, Machine Learning
