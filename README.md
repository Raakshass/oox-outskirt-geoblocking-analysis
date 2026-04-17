# COOX — Outskirt Booking Analysis & Geo-Blocking Strategy

**Tech GC 2026 | IIT Roorkee x COOX Problem Statement**

## Problem

COOX receives bookings from locations outside its serviceable areas — typically city outskirts or remote zones. These bookings are unfulfillable and get refunded, causing revenue leakage and poor customer experience.

## Objective

Analyze historical booking data to:
1. Identify geographical clusters of non-serviceable bookings
2. Extract pin codes for these problem zones
3. Recommend a tiered geo-blocking strategy to prevent future cancellations

## Key Results

| Metric | Value |
|--------|-------|
| Dataset | 557 bookings (546 after cleaning) |
| Clusters Found | 13 (DBSCAN, eps=2km) |
| Silhouette Score | 0.940 (strong cluster separation) |
| Pincodes Extracted | 67 unique (regex) + 13 centroids (reverse geocoding) |
| Combined Coverage | 30.8% of all non-serviceable bookings |

## Methodology

- **EDA**: City/state/address-type distributions, repeat offender analysis, cross-tabulation heatmaps
- **Geospatial**: Folium scatter maps, density heatmaps, city-level zoomed views (Bengaluru, Pune, Hyderabad)
- **Clustering**: DBSCAN with haversine distance, k-distance plot for parameter selection, silhouette score validation
- **Pincode Extraction**: Regex from address text + Nominatim reverse geocoding with automated verification
- **Geo-Blocking**: Tiered priority system (HIGH/MEDIUM/LOW/ISOLATED) with quantified impact analysis

## Repository Structure

```
├── README.md
├── COOX_Outskirt_Analysis_v2.ipynb          # Clean notebook (no outputs)
├── COOX_Outskirt_Analysis_v2_outputs.ipynb  # Notebook with execution outputs
├── data/
│   └── IIT Roorkee __ COOX - Raw Data.csv   # Source dataset
└── maps/                                     # Exported interactive maps (HTML)
    ├── scatter_map.html
    ├── heatmap.html
    ├── cluster_heatmap.html
    ├── blocking_zones.html
    ├── bengaluru_heatmap.html
    ├── pune_heatmap.html
    └── hyderabad_heatmap.html
```

## How to Run

1. Open `COOX_Outskirt_Analysis_v2.ipynb` in Google Colab
2. Upload the CSV from `data/` to Colab's file panel
3. Run all cells (`Runtime → Run All`)
4. Section 6 (reverse geocoding) takes ~3–5 minutes due to API rate limiting

## Tech Stack

- Python 3.10
- pandas, numpy, matplotlib, seaborn
- folium (interactive geospatial maps)
- scikit-learn (DBSCAN clustering)
- geopy (Nominatim reverse geocoding)

## Team

Tech GC 2026 Participants

## License

This project was created for the Tech GC 2026 competition at IIT Roorkee.
