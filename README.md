# 🌥️ Discovering Climate Zones Using Unsupervised Machine Learning on the NOAA Global Historical Climatology Network (GHCN) Dataset

Can a machine learning algorithm, given only raw weather observations from stations around the world, rediscover the climate zones that geographers have traditionally defined by hand?

This project explores that question using **unsupervised learning** on the **NOAA Global Historical Climatology Network Daily (GHCN-D)** dataset. With no climate labels, no continent identifiers, and no prior Köppen classes provided, the pipeline extracts station-level weather features, clusters stations using **KMeans**, and visualizes the resulting climate groupings on a world map.

## Project Highlights

* Processes large-scale weather data from NOAA GHCN-Daily
* Uses **Apache Spark** for distributed ingestion, feature engineering, and clustering
* Builds **Köppen-like proxy variables** from raw weather observations
* Compares clustering results with and without domain-informed feature engineering
* Exports clustered stations as a **GeoDataFrame/ESRI Shapefile** for GIS use
* Designed to run end-to-end on **Google Colab** or **Google Dataproc**

## Why This Project Matters

Climate zone classification is foundational to real-world decision-making.

* **Agriculture**: determines crop viability and growing conditions
* **Infrastructure**: informs building standards for cold, tropical, and arid environments
* **Environmental policy**: helps identify regions vulnerable to climate change

The project asks whether a data-driven pipeline can reconstruct climate structure automatically, and whether such a system could be retrained as climate patterns shift over time.

## Dataset

**Source:** [NOAA Global Historical Climatology Network Daily (GHCN-D)](https://www.ncei.noaa.gov/products/land-based-station/global-historical-climatology-network-daily)

**Key characteristics:**

* More than 160,000 weather stations globally
* Records spanning from 1763 to the present
* Daily observations updated continuously
* Raw archive stored as compressed station CSVs

**Variables used in this project:**

* `TMAX` — daily maximum temperature
* `TMIN` — daily minimum temperature
* `PRCP` — daily precipitation
* `SNWD` — snow depth

For development and demonstration, the pipeline processes a **sample of 200 stations** from a larger archive.

## Methodology

### 1. Data Ingestion

The NOAA archive is stored in Google Cloud Storage and accessed directly through Spark using the GCS Hadoop connector. The pipeline reads station files from a tar archive and samples stations reproducibly using a fixed random seed.

### 2. Parallel Feature Extraction

Each station is processed independently in parallel using Spark RDDs. This avoids repeatedly scanning the full archive and allows station-level feature computation to scale efficiently.

### 3. Feature Engineering

Instead of feeding raw daily measurements directly into the model, the pipeline computes seven climate-oriented features inspired by Köppen-Geiger logic.

#### Temperature Features

* **`heat_proxy`** — mean daily maximum temperature
* **`coldness_proxy`** — negative mean minimum temperature
* **`temp_range_proxy`** — average day-night temperature swing
* **`temp_spread_proxy`** — full observed temperature range

#### Moisture and Snow Features

* **`precip_wetness_proxy`** — mean daily precipitation
* **`precip_variability_proxy`** — precipitation variability
* **`snowiness_proxy`** — mean snow depth

All features are standardized before clustering.

### 4. Clustering

The project uses **KMeans** from Spark MLlib.

To choose the number of clusters, the pipeline evaluates values of **K from 2 to 6** and selects the model with the best **silhouette score**.

### 5. Spatial Output

Cluster assignments are exported with latitude and longitude so they can be visualized in GIS tools such as **QGIS** or **ArcGIS**.

## Results

The clustering pipeline produced climate groupings that reflect broad geographic structure:

* colder, snowier stations tend to cluster together
* warmer, drier stations tend to cluster together
* stations in similar climate regimes group even without labels

An important finding is that the **Köppen-like engineered features did not drastically outperform raw aggregate statistics** on the smaller sample. That suggests the handcrafted features may become more valuable at larger scale, where finer distinctions such as monsoon, oceanic, Mediterranean, and continental regimes may emerge more clearly.

## Repository Contents

A typical structure for this project includes:

```text
.
├── GHCN-climate-zones-discovery.ipynb
├── README.md
├── sample-data.csv
└── outputs/
```

## Notebook Overview

The main notebook demonstrates:

* Google Cloud authentication
* Spark and GCS connector setup
* archive ingestion and random station sampling
* parallel station processing with Spark RDDs
* Köppen-like feature engineering
* KMeans clustering and silhouette evaluation
* cluster summaries and visualizations
* shapefile export for GIS analysis

## Requirements

* Python 3.x
* Apache Spark
* PySpark
* Google Cloud Storage access
* GCS Hadoop connector
* pandas
* numpy
* geopandas
* matplotlib

## How to Run

1. Clone the repository.
2. Open `GHCN-climate-zones-discovery.ipynb` in Google Colab or a local Jupyter environment.
3. Authenticate with Google Cloud.
4. Set your **bucket name** and **GCP project ID**.
5. Run the notebook cells in order.

## Setup Notes

* In Colab, PySpark may need to be installed manually.
* Spark runs in local mode in Colab and in distributed mode on Dataproc.
* The GCS connector is required for reading and writing `gs://` paths directly.
* Station sampling is fixed by seed for reproducibility.

## Key Idea

The central idea behind this project is that meaningful climate structure can emerge from raw weather data alone. By combining large-scale data processing with unsupervised learning, the pipeline demonstrates a scalable way to rediscover climate zones without hand-labeled classes.

## Limitations

* The sample size is relatively small compared to the full NOAA archive
* Some climate distinctions may require a larger and more diverse station set
* Results depend on the quality and completeness of station records
* KMeans works best for roughly spherical clusters, which may not capture every climate boundary perfectly

## Future Work

* Scale the pipeline to a larger station sample
* Test additional clustering methods such as DBSCAN or hierarchical clustering
* Compare results more directly against Köppen-Geiger zones
* Improve spatial analysis with richer geospatial features
* Automate retraining as new observations arrive

## Conclusion

This project shows that unsupervised machine learning can rediscover broad climate structure from raw weather observations, and that Apache Spark makes the workflow practical at scale. Using distributed ingestion, parallel feature extraction, and MLlib clustering, the pipeline turns a massive climate archive into interpretable climate zones.

---

If you found this project useful, consider starring the repository and exploring the notebook for the full implementation.
