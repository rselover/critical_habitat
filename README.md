# USFWS Critical Habitat — NE/NY Similarity Screen

A geospatial analysis pipeline that fetches USFWS designated critical habitat polygons for New England and New York, embeds them using Google DeepMind's AlphaEarth satellite foundation model, and scores candidate land parcels by how closely their satellite signature resembles existing critical habitat.

## What It Does

1. **Fetches** finalized critical habitat designations from the USFWS ArcGIS REST API, clipped to the NE+NY region
2. **Embeds** each polygon as a 64-dimensional vector using AlphaEarth — trained on Sentinel-2, Landsat, and Sentinel-1 imagery
3. **Builds** a similarity distribution across all designated polygons
4. **Scores** a candidate parcel by cosine similarity to the regional centroid; parcels above 0.6 similarity are flagged for further review

A passing score means the land *looks like* existing critical habitat from the satellite's perspective — land cover, vegetation structure, moisture, and seasonal signal. It is a screening tool, not a regulatory determination.

## Setup

### Prerequisites

- [Miniconda or Anaconda](https://docs.conda.io/en/latest/miniconda.html)
- A Google Cloud account with access to the `alphaearth-496814` project
- `gcloud` CLI installed and authenticated:

```bash
gcloud auth application-default login
```

### Create the environment

```bash
conda env create -f environment.yml
conda activate critical_habitat
```

Or with pip:

```bash
pip install -r requirements.txt
```

### Launch the notebook

```bash
jupyter lab critical_habitat.ipynb
```

## Data Sources

| Source | Description |
|---|---|
| [USFWS Critical Habitat Portal](https://criticalhabitat.fws.gov/) | Federally designated critical habitat polygons via ArcGIS REST API |
| [AlphaEarth](https://deepmind.google/discover/blog/alphaearth-satellite-imagery-foundation-model/) | Google DeepMind satellite foundation model — 64-dim embeddings at 10 m resolution, served as COGs from GCS |

## Notes

- The USFWS API returns full multipolygon geometries that intersect the query bounding box — species like Piping Plover have habitat coast-to-coast. Geometries are clipped to NE+NY client-side after fetch.
- AlphaEarth embeddings are read from a requester-pays GCS bucket (`gs://alphaearth_foundations/`). GCS charges apply to the `alphaearth-496814` project.
- Bulk habitat extraction uses overview level 4 (~160 m) to minimize data transfer. Single parcel testing uses level 1 (~20 m) to ensure enough pixels within small polygon boundaries.
