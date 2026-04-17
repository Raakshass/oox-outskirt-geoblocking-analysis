# COOX Outskirt Booking Analysis and Geo-Blocking Strategy

**Tech GC 2026 | IIT Roorkee x COOX Problem Statement**

**[View Notebook on Google Colab](https://colab.research.google.com/drive/1bghYiCkpljnBE1RhdJr8Az2f7WkvOZ9g?usp=sharing)**

---

## Problem Statement

COOX is a chef-at-home services platform that receives bookings from locations outside its serviceable areas -- typically city outskirts, remote farmhouses, and industrial zones. These bookings are unfulfillable and result in full refunds, causing revenue leakage, operational overhead, and degraded customer experience.

**Objective:** Analyze 557 non-serviceable bookings to identify geographic patterns, extract actionable pincodes, and recommend a data-driven geo-blocking strategy.

---

## Key Results

| Metric | Value |
|--------|-------|
| Working Dataset | 546 bookings (11 dropped for missing coordinates) |
| Cities Covered | 21 across 14 states |
| Top 3 Problem Cities | Bengaluru (26.2%), Pune (17.2%), Hyderabad (12.5%) |
| Median Distance from City Center | 21.1 km |
| India-wide DBSCAN Clusters | 13 clusters, 95 bookings (17.4%) |
| City-Level Adaptive DBSCAN | 59 clusters, 414 bookings (75.8%) |
| Silhouette Score | 0.940 |
| Comprehensive Blocking Entries | 72 (13 cluster-based + 59 address-based) |
| Estimated Financial Impact | INR 2,265,900 |
| Projected Annual Savings (80%) | INR 1,812,720 |

---

## Methodology

```mermaid
flowchart TD
    A["Raw Data: 557 bookings"] --> B["Data Cleaning"]
    B --> C["546 working bookings"]
    C --> D["Exploratory Data Analysis"]
    C --> E["Distance from City Center"]
    C --> F["Geospatial Mapping"]
    C --> G["DBSCAN Clustering"]

    D --> D1["City/State/Address-type distributions"]
    D --> D2["Repeat offender detection: 68 addresses, 170 bookings"]

    E --> E1["Nominatim API: 21 city centers resolved dynamically"]
    E1 --> E2["Haversine distance per booking"]
    E2 --> E3["Geo-fence radius recommendation per city"]

    F --> F1["India-wide scatter and heatmaps"]
    F --> F2["City-level zoomed maps: Bengaluru, Pune, Hyderabad"]

    G --> G1["India-wide: 13 clusters, 17.4%"]
    G --> G2["City-level adaptive: 59 clusters, 75.8%"]

    G1 --> H["Pincode Extraction"]
    H --> H1["Regex: 67 unique pincodes from address text"]
    H --> H2["Reverse geocoding: 13 centroids + 66 locations"]
    H2 --> H3["Verification + auto-correction"]

    H3 --> I["Geo-Blocking Strategy"]
    H1 --> I
    I --> J["72 blocking entries + financial impact analysis"]

    style A fill:#fce4ec
    style J fill:#e8f5e9
    style G2 fill:#fff3e0
    style H3 fill:#e3f2fd
```

---

## Analysis Pipeline

```mermaid
graph LR
    subgraph Input
        CSV["557 rows x 10 cols"]
    end

    subgraph Processing
        CLEAN["Cleaning + Feature Engineering"]
        EDA["EDA + Distance Analysis"]
        GEO["7 Interactive Maps"]
        CLUST["Dual DBSCAN Clustering"]
        PIN["Pincode Extraction + Verification"]
    end

    subgraph Output
        BLOCK["72 Blocking Entries"]
        FIN["INR 2.27M Impact"]
        MAPS["4 Downloadable HTML Maps"]
    end

    CSV --> CLEAN --> EDA
    CLEAN --> GEO
    CLEAN --> CLUST --> PIN --> BLOCK --> FIN
    GEO --> MAPS

    style CSV fill:#fce4ec
    style BLOCK fill:#e8f5e9
    style FIN fill:#e8f5e9
    style MAPS fill:#e8f5e9
```

---

## Clustering Strategy

Two levels of DBSCAN are applied to maximize geographic coverage:

```mermaid
flowchart LR
    subgraph India["India-wide DBSCAN"]
        I1["eps=2km, min_samples=5"]
        I1 --> I2["13 clusters, 17.4% coverage"]
    end

    subgraph City["City-Level Adaptive DBSCAN"]
        C1["eps=P75 k-distance per city"]
        C1 --> C2["59 clusters, 75.8% coverage"]
    end

    I2 -->|"4.4x improvement"| C2

    style I2 fill:#fce4ec
    style C2 fill:#e8f5e9
```

| City | eps (km) | Clusters | Coverage |
|------|----------|----------|----------|
| Bengaluru | 4.1 | 14 | 79.0% |
| Pune | 4.0 | 7 | 74.5% |
| Hyderabad | 7.4 | 7 | 79.4% |
| Chennai | 3.6 | 9 | 75.8% |
| Ahmedabad | 4.1 | 3 | 81.6% |
| Jaipur | 8.7 | 3 | 84.0% |
| Kolkata | 8.9 | 3 | 88.9% |

---

## Pincode Extraction Pipeline

```mermaid
flowchart TD
    subgraph Method1["Regex Extraction"]
        A1["Address text"] -->|"Pattern: \\d{6}"| A2["94 matches, 67 unique pincodes"]
    end

    subgraph Method2["Reverse Geocoding"]
        B1["13 cluster centroids"] -->|Nominatim| B2["13 pincodes"]
        B3["66 unique cluster locations"] -->|Nominatim| B4["66 pincodes"]
    end

    subgraph Verification
        B2 --> C1["State-prefix validation"]
        C1 -->|"12/13 OK"| C2["Verified"]
        C1 -->|"654321 is Kerala, not Pune"| C3["Auto-corrected to 412101"]
        C3 --> C2
    end

    A2 --> D["72 actionable pincode entries"]
    C2 --> D
    B4 --> D

    style C3 fill:#fff3e0
    style D fill:#e8f5e9
```

---

## Geo-Fence Recommendations

City centers resolved dynamically via Nominatim. The 25th percentile of booking distances used as the recommended blocking radius (prevents 75% of outskirt bookings).

| City | Bookings | Median Distance | Recommended Geo-fence |
|------|----------|----------------|-----------------------|
| Bengaluru | 143 | 22.1 km | 19 km |
| Pune | 94 | 19.5 km | 15 km |
| Hyderabad | 68 | 26.2 km | 22 km |
| Chennai | 66 | 26.5 km | 21 km |
| Ahmedabad | 38 | 15.8 km | 14 km |
| Jaipur | 25 | 20.8 km | 16 km |

---

## Blocking Priority Tiers

```mermaid
graph TD
    subgraph HIGH["HIGH -- Block Immediately"]
        H1["Clusters with 10+ bookings"]
        H2["Pune 412101, Bengaluru 562125, Chennai 600130"]
    end

    subgraph MEDIUM["MEDIUM -- Block Recommended"]
        M1["Clusters with 5-9 bookings"]
        M2["Surat, Ahmedabad, Kolkata zones"]
    end

    subgraph LOW["LOW -- Monitor"]
        L1["Isolated pincodes, 1-4 bookings each"]
    end

    HIGH --> MEDIUM --> LOW

    style HIGH fill:#ffcdd2
    style MEDIUM fill:#fff9c4
    style LOW fill:#c8e6c9
```

---

## Financial Impact

| Metric | Value |
|--------|-------|
| Revenue at Risk | INR 2,184,000 |
| Processing Waste | INR 81,900 |
| Total Impact | INR 2,265,900 |
| Projected Annual Savings (80% prevention) | INR 1,812,720 |
| Projected Monthly Savings | INR 151,060 |

---

## Repository Structure

```
oox-outskirt-geoblocking-analysis/
|-- README.md
|-- COOX_Outskirt_Analysis_final.ipynb
|-- data/
    |-- IIT Roorkee __ COOX - Raw Data.csv
```

---

## Setup and Execution

1. Open the [Google Colab notebook](https://colab.research.google.com/drive/1bghYiCkpljnBE1RhdJr8Az2f7WkvOZ9g?usp=sharing)
2. Upload the CSV from `data/` to the Colab file panel
3. `Runtime` > `Run All`
4. Total runtime: 5-8 minutes (API rate-limited sections)

### Dependencies

pandas, numpy, matplotlib, seaborn, folium, scikit-learn, geopy

---

## Notebook Sections

| # | Section | Key Output |
|---|---------|-----------|
| 1 | Setup and Data Loading | 557 rows x 10 columns |
| 2 | Data Cleaning | 546 clean bookings |
| 3 | Exploratory Data Analysis | City/state distributions, repeat offenders |
| 3.1 | Distance from City Center | Median 21.1 km, geo-fence radii |
| 4 | Geospatial Visualization | 7 interactive maps |
| 5 | DBSCAN Clustering | 13 India-wide + 59 city-level clusters |
| 6 | Pincode Extraction | 72 verified pincodes |
| 7 | Geo-Blocking Recommendation | Tiered blocking list, financial impact |
| 8 | Limitations and Future Work | Enhancement roadmap |
| 9 | Executive Summary | Consolidated findings |

---

## Limitations

- No temporal data in dataset (no trend analysis possible)
- Financial impact uses industry-average estimates
- Free geocoding API has occasional inaccuracies (auto-corrected where detected)

## Future Work

- Integrate with COOX internal API for real-time booking validation
- Replace Nominatim with India Post official pincode database
- Build operational dashboard for the operations team
- A/B test geo-blocking rules to measure actual prevention rates

---

*Tech GC 2026, IIT Roorkee*
