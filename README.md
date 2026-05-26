# USFWS Critical Habitat — NE/NY Similarity Screen

A geospatial analysis pipeline that fetches USFWS designated critical habitat polygons for New England and New York, embeds them using Google DeepMind's AlphaEarth satellite foundation model, and scores candidate land parcels by how closely their satellite signature resembles existing critical habitat.

## What It Does

1. **Fetches** finalized critical habitat designations from the USFWS ArcGIS REST API, clipped to the NE+NY region
2. **Embeds** each polygon as a 64-dimensional vector using AlphaEarth — trained on Sentinel-2, Landsat, and Sentinel-1 imagery
3. **Builds** a similarity distribution across all designated polygons
4. **Scores** a candidate parcel by cosine similarity to the regional centroid; parcels above 0.6 similarity are flagged for further review

A passing score means the land *looks like* existing critical habitat from the satellite's perspective — land cover, vegetation structure, moisture, and seasonal signal. It is a screening tool, not a regulatory determination.

## Setup

### 1. Google Cloud Account

AlphaEarth embeddings are stored in a requester-pays Google Cloud Storage bucket. You need your own GCP project to cover the read costs.

1. Create a Google account if you don't have one
2. Go to [console.cloud.google.com](https://console.cloud.google.com) and create a new project
3. Enable billing on the project — GCS egress costs apply when reading the embedding tiles (reads are small at overview levels; expect cents per run, not dollars)
4. Enable the **Cloud Storage API** for the project: `APIs & Services → Enable APIs → Cloud Storage`
5. Install the [gcloud CLI](https://cloud.google.com/sdk/docs/install) and authenticate:

```bash
gcloud auth application-default login
```

6. Open the notebook and set your project ID in the AlphaEarth setup cell:

```python
GCS_PROJECT = "your-gcp-project-id"
```

### 2. Python Environment

**With conda (recommended):**

```bash
conda env create -f environment.yml
conda activate critical_habitat
```

**With pip:**

```bash
pip install -r requirements.txt
```

### 3. Launch the Notebook

```bash
jupyter lab critical_habitat.ipynb
```

## Data Sources

| Source | Description |
|---|---|
| [USFWS Critical Habitat Portal](https://criticalhabitat.fws.gov/) | Federally designated critical habitat polygons via ArcGIS REST API — no account required |
| [AlphaEarth](https://deepmind.google/discover/blog/alphaearth-satellite-imagery-foundation-model/) | Google DeepMind satellite foundation model — 64-dim embeddings at 10 m resolution, served as COGs from GCS |

## Notes

- The USFWS API returns full multipolygon geometries that intersect the query bounding box — species like Piping Plover have habitat coast-to-coast. Geometries are clipped to NE+NY client-side after fetch.
- AlphaEarth embeddings are read from a requester-pays GCS bucket (`gs://alphaearth_foundations/`). Read costs are billed to whichever GCP project you set in `GCS_PROJECT`.
- Bulk habitat extraction uses overview level 4 (~160 m) to minimize data transfer. Single parcel testing uses level 1 (~20 m) to ensure enough pixels within small polygon boundaries.
