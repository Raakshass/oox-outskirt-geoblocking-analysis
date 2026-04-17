# COOX Outskirt Booking Analysis and Geo-Blocking Strategy

**Tech GC 2026 | IIT Roorkee x COOX Problem Statement**

---

## Table of Contents

- [Problem Statement](#problem-statement)
- [Executive Summary](#executive-summary)
- [Key Results](#key-results)
- [Methodology](#methodology)
- [Pipeline Architecture](#pipeline-architecture)
- [Data Flow](#data-flow)
- [Distance Analysis](#distance-analysis)
- [Clustering Strategy](#clustering-strategy)
- [Pincode Extraction Pipeline](#pincode-extraction-pipeline)
- [Geo-Blocking Recommendation](#geo-blocking-recommendation)
- [Financial Impact](#financial-impact)
- [Repository Structure](#repository-structure)
- [Setup and Execution](#setup-and-execution)
- [Tech Stack](#tech-stack)
- [Limitations and Future Work](#limitations-and-future-work)

---

## Problem Statement

COOX is a chef-at-home services platform that receives bookings from locations outside its serviceable areas -- typically city outskirts, remote farmhouses, and industrial zones. These bookings are unfulfillable and result in full refunds, causing:

- Revenue leakage from processing failed orders
- Operational overhead from customer support handling refund requests
- Degraded customer experience from order cancellations

**Objective:** Analyze historical non-serviceable booking data to identify geographic patterns, extract actionable pincodes, and recommend a data-driven geo-blocking strategy to prevent future cancellations.

---

## Executive Summary

This analysis processes 557 non-serviceable (fully refunded) bookings across 14 Indian states and 21 cities. Through a multi-stage pipeline combining exploratory analysis, geospatial visualization, density-based clustering, automated pincode extraction, and distance-from-center analysis, we deliver:

1. A comprehensive understanding of where and why bookings fail geographically
2. A quantified definition of "outskirt" using dynamic city-center resolution
3. A tiered geo-blocking recommendation covering 72 pincode entries
4. Per-city geo-fence radius recommendations derived from booking distance distributions
5. Estimated financial impact of INR 2.27M with projected annual savings of INR 1.81M

---

## Key Results

| Metric | Value |
|--------|-------|
| Raw Dataset | 557 bookings, 10 columns |
| Working Dataset | 546 bookings (11 dropped for missing coordinates) |
| Cities Covered | 21 across 14 states |
| Top 3 Problem Cities | Bengaluru (26.2%), Pune (17.2%), Hyderabad (12.5%) |
| Median Distance from City Center | 21.1 km |
| Maximum Distance | 179.3 km |
| India-wide DBSCAN Clusters | 13 clusters, 95 bookings (17.4%) |
| City-Level Adaptive DBSCAN | 59 clusters, 414 bookings (75.8%) |
| Silhouette Score | 0.940 (excellent separation) |
| Pincodes Extracted (Regex) | 67 unique pincodes from 94 bookings |
| Pincodes Extracted (Geocoding) | 13 cluster centroids + 66 individual locations |
| Comprehensive Blocking Entries | 72 (13 clusters + 59 isolated pincodes) |
| Total Bookings Addressable | 168 / 546 (30.8%) |
| Estimated Revenue Impact | INR 2,265,900 |
| Projected Annual Savings | INR 1,812,720 (at 80% prevention rate) |

---

## Methodology

```mermaid
flowchart TD
    A["Raw Data<br/>557 bookings"] --> B["Data Cleaning<br/>Column standardization<br/>Missing value handling<br/>Type conversion"]
    B --> C["Working Dataset<br/>546 bookings"]
    C --> D["Exploratory Data Analysis"]
    C --> E["Distance Analysis"]
    C --> F["Geospatial Visualization"]
    C --> G["Cluster Identification"]

    D --> D1["City/State/Address-type<br/>distributions"]
    D --> D2["Cross-tabulation<br/>heatmaps"]
    D --> D3["Repeat offender<br/>detection"]

    E --> E1["Dynamic city center<br/>resolution via Nominatim"]
    E1 --> E2["Haversine distance<br/>computation"]
    E2 --> E3["Distance distribution<br/>plots"]
    E3 --> E4["Per-city geo-fence<br/>radius recommendation"]

    F --> F1["India-wide scatter<br/>and heatmaps"]
    F --> F2["City-level zoomed<br/>heatmaps"]

    G --> G1["India-wide DBSCAN<br/>eps=2km, 13 clusters"]
    G --> G2["City-Level Adaptive<br/>DBSCAN, 59 clusters"]

    G1 --> H["Pincode Extraction"]
    H --> H1["Regex from<br/>address text"]
    H --> H2["Reverse geocoding<br/>cluster centroids"]
    H --> H3["Reverse geocoding<br/>individual locations"]
    H2 --> H4["Pincode Verification<br/>and Auto-Correction"]

    H4 --> I["Geo-Blocking Strategy"]
    H1 --> I
    H3 --> I

    I --> I1["Tiered Priority<br/>Assignment"]
    I1 --> I2["Comprehensive<br/>Blocking List"]
    I2 --> I3["Financial Impact<br/>Estimation"]
    I3 --> J["Final Deliverables<br/>72 blocking entries<br/>Interactive maps<br/>Geo-fence radii"]

    style A fill:#fce4ec
    style J fill:#e8f5e9
    style G2 fill:#fff3e0
    style H4 fill:#e3f2fd
    style E4 fill:#f3e5f5
```

---

## Pipeline Architecture

```mermaid
graph LR
    subgraph Input
        CSV["Raw CSV<br/>557 rows x 10 cols"]
    end

    subgraph Preprocessing
        CLEAN["Column Standardization<br/>Missing Value Handling<br/>Type Coercion<br/>Duplicate Check"]
    end

    subgraph Analysis
        EDA["EDA Module<br/>City/State distributions<br/>Address-type breakdown<br/>Repeat offenders (68 addresses)"]
        DIST["Distance Module<br/>Nominatim city-center lookup<br/>Haversine computation<br/>Geo-fence recommendation"]
        GEO["Geospatial Module<br/>7 interactive maps<br/>India-wide and city-level<br/>Scatter + Heatmap + Cluster"]
        CLUST["Clustering Module<br/>DBSCAN (India-wide + City-level)<br/>k-distance parameter selection<br/>Silhouette validation"]
        PIN["Pincode Module<br/>Regex extraction<br/>Reverse geocoding<br/>State-prefix validation<br/>Auto-correction fallback"]
    end

    subgraph Output
        BLOCK["Blocking Strategy<br/>72 entries, 3 priority tiers"]
        FIN["Financial Impact<br/>INR 2.27M at risk"]
        MAPS["Interactive Maps<br/>4 HTML exports"]
    end

    CSV --> CLEAN --> EDA
    CLEAN --> DIST
    CLEAN --> GEO
    CLEAN --> CLUST
    CLUST --> PIN
    PIN --> BLOCK
    BLOCK --> FIN
    GEO --> MAPS

    style CSV fill:#fce4ec
    style BLOCK fill:#e8f5e9
    style FIN fill:#e8f5e9
    style MAPS fill:#e8f5e9
```

---

## Data Flow

```mermaid
flowchart TB
    subgraph Stage1["Stage 1: Data Ingestion"]
        RAW["557 raw bookings<br/>All refunded_full"]
        RAW -->|Drop 11 missing coords| CLEAN["546 clean bookings<br/>0 duplicates"]
    end

    subgraph Stage2["Stage 2: Feature Engineering"]
        CLEAN -->|Nominatim API| CENTER["City Center<br/>Coordinates<br/>21 cities resolved"]
        CENTER -->|Haversine| DISTANCE["dist_from_center_km<br/>Median: 21.1 km<br/>Range: 1.3 - 179.3 km"]
        CLEAN -->|Regex| REGEX_PIN["pincode_from_address<br/>94 bookings matched<br/>67 unique pincodes"]
    end

    subgraph Stage3["Stage 3: Clustering"]
        CLEAN -->|DBSCAN eps=2km| INDIA["India-wide<br/>13 clusters<br/>95 bookings (17.4%)"]
        CLEAN -->|DBSCAN adaptive eps| CITY["City-level<br/>59 clusters<br/>414 bookings (75.8%)"]
    end

    subgraph Stage4["Stage 4: Pincode Resolution"]
        INDIA -->|Centroids| GEOCODE["Reverse Geocoding<br/>13 centroid pincodes<br/>66 individual pincodes"]
        GEOCODE -->|State prefix check| VERIFY["Verification<br/>12/13 OK<br/>1 auto-corrected"]
    end

    subgraph Stage5["Stage 5: Output"]
        VERIFY --> FINAL["72 Blocking Entries<br/>13 cluster-based<br/>59 address-based"]
        REGEX_PIN --> FINAL
        FINAL --> IMPACT["INR 2,265,900<br/>total impact"]
    end

    style RAW fill:#fce4ec
    style FINAL fill:#e8f5e9
    style CITY fill:#fff3e0
    style VERIFY fill:#e3f2fd
```

---

## Distance Analysis

The distance analysis provides a quantitative, data-driven definition of "outskirt" by dynamically resolving each city's official center coordinates through Nominatim (OpenStreetMap) and computing the haversine distance for every booking.

### How It Works

```mermaid
sequenceDiagram
    participant NB as Notebook
    participant NOM as Nominatim API
    participant DF as DataFrame

    NB->>NB: Extract 21 unique city names
    loop For each city
        NB->>NOM: geocode("City, India")
        alt Success
            NOM-->>NB: (latitude, longitude)
        else API failure
            NB->>DF: Use median of city bookings
        end
    end
    NB->>DF: Compute haversine(city_center, booking_coords)
    NB->>NB: Add dist_from_center_km column
    NB->>NB: Generate distribution plots
    NB->>NB: Calculate P25 as geo-fence radius
```

### Recommended Geo-Fence Radii

The 25th percentile of outskirt booking distances is used as the recommended geo-fence radius. Blocking bookings beyond this radius would prevent approximately 75% of non-serviceable bookings for each city.

| City | Bookings | Min Distance | Median Distance | Suggested Geo-fence |
|------|----------|-------------|-----------------|---------------------|
| Bengaluru | 143 | 10.3 km | 22.1 km | 19 km |
| Pune | 94 | 6.5 km | 19.5 km | 15 km |
| Hyderabad | 68 | 15.4 km | 26.2 km | 22 km |
| Chennai | 66 | 13.2 km | 26.5 km | 21 km |
| Ahmedabad | 38 | 10.1 km | 15.8 km | 14 km |
| Jaipur | 25 | 12.5 km | 20.8 km | 16 km |

---

## Clustering Strategy

### Dual DBSCAN Approach

Two levels of DBSCAN clustering are applied to maximize coverage:

```mermaid
flowchart LR
    subgraph India["India-wide DBSCAN"]
        I1["eps = 2 km<br/>min_samples = 5<br/>metric = haversine"]
        I1 --> I2["13 clusters<br/>95 bookings<br/>17.4% coverage"]
    end

    subgraph City["City-Level Adaptive DBSCAN"]
        C1["eps = P75 of k-distance<br/>per city (3-15 km)<br/>min_samples = 3"]
        C1 --> C2["59 clusters<br/>414 bookings<br/>75.8% coverage"]
    end

    I2 -->|"4.4x improvement"| C2

    style I2 fill:#fce4ec
    style C2 fill:#e8f5e9
```

### Why DBSCAN

- Does not require pre-specifying the number of clusters (unlike K-Means)
- Handles arbitrary geographic cluster shapes (outskirts are not circular)
- Identifies noise points (isolated bookings that do not belong to any cluster)
- Uses haversine distance metric for accurate geographic distance on Earth's surface

### Parameter Selection

- **eps (India-wide):** Selected via k-distance plot elbow method at 2 km
- **eps (City-level):** Adaptive, computed as the 75th percentile of k-distance per city, clamped between 3 km and 15 km
- **min_samples (India-wide):** 5 (standard geographic clustering minimum)
- **min_samples (City-level):** 3 (lower threshold to capture smaller city-specific patterns)
- **Silhouette Score:** 0.940 (indicates strong cluster cohesion and separation)

### City-Level DBSCAN Results

| City | eps (km) | Clusters | Clustered Bookings | Total | Coverage |
|------|----------|----------|--------------------|-------|----------|
| Bengaluru | 4.1 | 14 | 113 | 143 | 79.0% |
| Pune | 4.0 | 7 | 70 | 94 | 74.5% |
| Hyderabad | 7.4 | 7 | 54 | 68 | 79.4% |
| Chennai | 3.6 | 9 | 50 | 66 | 75.8% |
| Ahmedabad | 4.1 | 3 | 31 | 38 | 81.6% |
| Jaipur | 8.7 | 3 | 21 | 25 | 84.0% |
| Kolkata | 8.9 | 3 | 16 | 18 | 88.9% |

---

## Pincode Extraction Pipeline

```mermaid
flowchart TD
    subgraph Method1["Method 1: Regex Extraction"]
        A1["Address text field"] -->|"Pattern: \\d{6}"| A2["94 bookings matched"]
        A2 --> A3["67 unique pincodes"]
    end

    subgraph Method2["Method 2: Reverse Geocoding"]
        B1["13 cluster centroids"] -->|Nominatim API| B2["13 centroid pincodes"]
        B3["66 unique locations<br/>in hotspot clusters"] -->|Nominatim API| B4["66 location pincodes"]
    end

    subgraph Verification["Verification Layer"]
        B2 --> C1["State-prefix validation"]
        C1 -->|"12/13 passed"| C2["Verified pincodes"]
        C1 -->|"1/13 failed<br/>654321 is Kerala pin<br/>not Pune"| C3["Auto-correction"]
        C3 -->|"Fallback to most common<br/>regex-extracted pin<br/>from cluster"| C4["654321 corrected<br/>to 412101"]
        C4 --> C2
    end

    A3 --> D["Combined Output"]
    C2 --> D
    B4 --> D
    D --> E["72 actionable<br/>pincode entries"]

    style C3 fill:#fff3e0
    style C4 fill:#e8f5e9
    style E fill:#e8f5e9
```

---

## Geo-Blocking Recommendation

### Priority Tiers

```mermaid
graph TD
    subgraph HIGH["HIGH PRIORITY -- Block Immediately"]
        H1["Clusters with 10+ bookings"]
        H2["Pune Cluster 2: Pin 412101 (13 bookings)"]
        H3["Bengaluru Cluster 4: Pin 562125 (10 bookings)"]
        H4["Chennai Cluster 3: Pin 600130 (10 bookings)"]
    end

    subgraph MEDIUM["MEDIUM PRIORITY -- Block Recommended"]
        M1["Clusters with 5-9 bookings"]
        M2["Surat 394270, Bengaluru 560049<br/>Ahmedabad 382722, and 7 more"]
    end

    subgraph LOW["LOW PRIORITY -- Monitor"]
        L1["Isolated pincodes with 1-4 bookings"]
        L2["59 address-extracted pincodes"]
    end

    HIGH --> MEDIUM --> LOW

    style HIGH fill:#ffcdd2
    style MEDIUM fill:#fff9c4
    style LOW fill:#c8e6c9
```

### Strategy Overview

| Strategy | Method | Clusters/Entries | Bookings Covered |
|----------|--------|-----------------|------------------|
| Zone-Based Blocking | India-wide DBSCAN hotspots | 13 cluster pincodes | 95 (17.4%) |
| Pincode-Level Blocking | Address regex extraction | 59 isolated pincodes | 73 (13.4%) |
| Combined | Both strategies merged | 72 total entries | 168 (30.8%) |
| City-Level Insight | City-adaptive DBSCAN | 59 local clusters | 414 (75.8%) |

---

## Financial Impact

```mermaid
pie title Revenue Impact Distribution
    "Clustered Hotspot Bookings (95)" : 380000
    "Isolated Pincode Bookings (73)" : 292000
    "Remaining Uncovered (378)" : 1512000
```

| Metric | Value |
|--------|-------|
| Average Order Value (estimated) | INR 4,000 |
| Processing Cost per Failed Booking | INR 150 |
| Total Revenue at Risk | INR 2,184,000 |
| Total Processing Waste | INR 81,900 |
| **Total Estimated Impact** | **INR 2,265,900** |
| Projected Annual Savings (80% prevention) | INR 1,812,720 |
| Projected Monthly Savings | INR 151,060 |

---

## Repository Structure

```
oox-outskirt-geoblocking-analysis/
|-- README.md                               # This file
|-- COOX_Outskirt_Analysis_final.ipynb      # Complete notebook with execution outputs
|-- data/
|   |-- IIT Roorkee __ COOX - Raw Data.csv  # Source dataset (557 rows)
|-- maps/                                    # Exported interactive maps
    |-- 01_scatter_map.html                  # All booking locations
    |-- 02_density_heatmap.html              # Booking density visualization
    |-- 03_cluster_heatmap.html              # Cluster-only density
    |-- 04_blocking_zones.html               # Final blocking recommendation
```

---

## Setup and Execution

### Prerequisites

- Python 3.10 or higher
- Google Colab (recommended) or Jupyter Notebook

### Dependencies

```
pandas
numpy
matplotlib
seaborn
folium
scikit-learn
geopy
```

### Running the Notebook

1. Open `COOX_Outskirt_Analysis_final.ipynb` in Google Colab
2. Upload the CSV file from `data/` to the Colab file panel
3. Select `Runtime` then `Run All`
4. Wait approximately 5-8 minutes for completion (API rate-limited sections)

### Runtime Notes

- Section 3.1 (City Center Resolution): Calls Nominatim API for 21 cities with 1.5s rate limiting
- Section 6 (Pincode Extraction): Calls Nominatim API for 13 centroids + 66 individual locations
- All other sections execute in under 10 seconds each

---

## Tech Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| Data Processing | pandas, numpy | Cleaning, aggregation, feature engineering |
| Visualization | matplotlib, seaborn | Static charts and distribution plots |
| Geospatial Maps | folium | Interactive scatter maps, heatmaps, cluster maps |
| Clustering | scikit-learn (DBSCAN) | Density-based geographic clustering |
| Geocoding | geopy (Nominatim) | City center resolution, reverse geocoding |
| Distance | Custom haversine | Haversine formula for geographic distance |

---

## Limitations and Future Work

### Current Limitations

- **No temporal data:** The dataset lacks timestamps, preventing trend analysis and seasonality detection
- **No revenue column:** Financial impact relies on industry-average estimates rather than actual booking values
- **Free geocoding API:** Nominatim rate limits and occasional inaccuracies (e.g., the 654321 pincode issue, auto-corrected)
- **Static dataset:** Analysis represents a snapshot; production would require continuous ingestion

### Recommended Enhancements

- Integrate with COOX internal API for real-time booking validation
- Replace Nominatim with India Post official pincode database for verification
- Add temporal dimension when timestamp data becomes available
- Implement automated model retraining pipeline as new booking data accumulates
- Build a dashboard (Streamlit or Metabase) for operations team visibility
- A/B test geo-blocking rules to measure actual prevention rates

---

## Notebook Sections

| Section | Description | Key Output |
|---------|-------------|-----------|
| 1. Setup and Data Loading | Import libraries, load CSV | 557 rows x 10 columns |
| 2. Data Cleaning | Column standardization, missing values, type conversion | 546 clean bookings |
| 3. Exploratory Data Analysis | City/state/address distributions, repeat offenders | Top 20 problem areas |
| 3.1 Distance Analysis | Dynamic city-center resolution, haversine distance | Median 21.1 km, geo-fence radii |
| 4. Geospatial Visualization | Scatter maps, density heatmaps, city-level zoomed maps | 7 interactive maps |
| 5. Cluster Identification | India-wide DBSCAN, k-distance plot, silhouette validation | 13 clusters, 0.940 score |
| 5.2 City-Level DBSCAN | Adaptive per-city clustering | 59 clusters, 75.8% coverage |
| 6. Pincode Extraction | Regex + reverse geocoding + verification + auto-correction | 72 actionable pincodes |
| 7. Geo-Blocking Recommendation | Tiered priority table, comprehensive blocking list | 168 bookings addressable |
| 7.1 Isolated Locations | Non-clustered booking analysis | 451 isolated bookings |
| 8. Limitations and Future Work | Scope constraints and enhancement roadmap | 6 recommendations |
| 9. Executive Summary | Consolidated findings and recommendations | Final deliverable |

---

*This analysis was conducted as part of the Tech GC 2026 competition at IIT Roorkee.*
