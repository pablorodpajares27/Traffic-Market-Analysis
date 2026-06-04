# Architecture

This document describes the architecture behind the BTS O&D Market Analysis project.

The goal of the project is to transform raw BTS Origin & Destination survey files into structured, analysis-ready tables that can support airline market analysis, competitive diagnostics, connectivity assessment and route opportunity screening.

The architecture is organized as a layered data pipeline. Each layer has a specific analytical role and keeps a clear separation between raw data, modeled travel records, aggregated market cubes and final business-facing outputs.

---

## 1. Architecture overview

```text
Raw BTS O&D survey files
        ↓
Program 1 — Ingestion and Parquet standardization
        ↓
Program 2 — Itineraries and Segments modeling
        ↓
Program 3 — YAML-driven aggregation engine
        ↓
Program 4 — Analysis-ready market tables
        ↓
Visual analysis / BI-ready outputs
```

The pipeline follows a progressive transformation approach:

| Layer                | Main output                 | Purpose                                                                   |
| -------------------- | --------------------------- | ------------------------------------------------------------------------- |
| Raw source           | BTS DB1C / DB1B files       | Original survey files by reporting period.                                |
| Standardized storage | Period-based Parquet files  | Clean, auditable and column-consistent storage layer.                     |
| Travel model         | Itineraries and segments    | Analytical representation of O&D journeys and individual travel segments. |
| Aggregated cubes     | Market-level grouped tables | Reusable market, carrier, airport, booking and connectivity cubes.        |
| Analysis layer       | Final analysis tables       | Business-ready outputs for market diagnostics and opportunity analysis.   |

---

## 2. Source data layer

The project starts from BTS O&D survey files, mainly:

* **DB1C monthly data**
* **DB1B quarterly data**

These files are not convenient to analyze directly because they are large, period-based, structurally complex and may contain wide positional fields, embedded headers or different schemas depending on the source type.

The raw data layer is kept separate from all derived outputs. This makes the pipeline easier to audit, rerun and extend.

---

## 3. Standardized Parquet layer

The first processing layer converts the raw source files into standardized Parquet outputs.

This layer is responsible for:

* detecting whether a source file belongs to DB1C or DB1B;
* detecting the reporting year, month or quarter;
* extracting compressed source files;
* archiving the original input;
* converting tabular files into Parquet;
* adding technical metadata such as source file, period and dataset type;
* standardizing column layout where possible.

The output of this layer becomes the stable storage foundation for the rest of the project.

Example folders:

```text
Files by month/
Files by quarter/
Archive/
Logs/
```

---

## 4. Itineraries and segments layer

The second layer creates two different analytical representations of the travel data:

```text
Itineraries = one row per complete O&D journey
Segments    = one row per coupon / segment inside a journey
```

This separation is important because market demand and network movement are not the same analytical grain.

---

## 5. Why separate itineraries and segments?

The **itinerary layer** represents what the passenger bought.

It is used to analyze:

* origin and final destination;
* O&D market size;
* fare and revenue;
* number of segments;
* nonstop vs connecting journeys;
* first marketing or operating carrier;
* purchase window where available.

The **segment layer** represents how the journey was physically structured.

It is used to analyze:

* individual segment origin and destination;
* intermediate airports;
* segment carriers;
* segment distance;
* dwell time after a segment;
* connection structure.

Keeping these layers separate avoids mixing passenger demand with network movement. This is especially important in O&D analysis, where a passenger may buy one market but travel through multiple intermediate points.

---

## 6. Aggregated market cube layer

The third layer builds reusable analytical cubes from the itinerary and segment tables.

These cubes are generated from YAML configuration files. The YAML files define:

* input source;
* period type;
* grouping dimensions;
* metrics;
* post-aggregation calculations;
* output table names.

Typical aggregation dimensions include:

* O&D city market;
* airport pair;
* WAC pair;
* carrier;
* itinerary type;
* number of segments;
* purchase window.

Typical metrics include:

* passengers;
* total amount;
* average fare;
* nonstop and connecting passengers;
* market share;
* HHI;
* weighted average distance;
* yield;
* growth metrics.

This layer is designed to make the analytical logic configurable without rewriting the processing engine for every table.

---

## 7. Analysis-ready layer

The final layer combines the aggregated cubes into compact, business-facing tables.

These outputs are designed for market analysis and visual exploration, rather than raw data processing.

Main final tables include:

| Table                         | Description                                                             |
| ----------------------------- | ----------------------------------------------------------------------- |
| `market_core_summary`         | Core O&D market view for passengers, revenue, fares, yield and trends.  |
| `market_competition_summary`  | Carrier leadership, market share, HHI and concentration indicators.     |
| `market_connectivity_summary` | Nonstop vs connecting mix, segment complexity and friction metrics.     |
| `market_booking_summary`      | Purchase-window mix, booking behavior and fare by booking timing.       |
| `market_airport_summary`      | Airport-pair drill-down for route-level traffic and fare analysis.      |
| `market_wac_summary`          | Regional traffic flows and macro geographic market view.                |
| `market_seasonality_summary`  | Seasonal demand patterns, peak quarter and quarterly variation.         |
| `market_opportunity_summary`  | Strategic opportunity scoring and market prioritization.                |
| `dim_period`                  | Time dimension for filtering, period alignment and reporting structure. |

---

## 8. Design principles

The project follows several design principles.

### Separation of layers

Each processing stage has a specific role. Raw files, modeled records, aggregated cubes and final analysis tables are not mixed.

### Clear analytical grain

Itineraries, segments, market cubes and final summaries represent different levels of detail. Keeping these grains separate improves interpretability and prevents incorrect calculations.

### Reproducibility

The pipeline can be rerun when new monthly or quarterly BTS files are added.

### Auditability

Original files are archived, outputs are written by period, and processing logs are generated.

### Configurable aggregation

The aggregation layer is driven by YAML configuration, allowing new market cubes to be added without redesigning the entire engine.

### BI-ready outputs

The final tables are designed to reduce the amount of transformation required in downstream visualization tools.

---

## 9. Monthly and quarterly outputs

The project supports both monthly and quarterly outputs.

Monthly data is useful for:

* tactical monitoring;
* recent demand changes;
* short-term trend detection;
* booking-window analysis where available.

Quarterly data is useful for:

* structural trends;
* seasonality;
* longer-term market evaluation;
* route opportunity screening.

Monthly and quarterly tables should generally be analyzed separately because they represent different period grains.

---

## 10. Final architecture summary

The architecture converts complex BTS O&D survey data into a structured aviation market analysis layer.

```text
Raw survey files
    → standardized Parquet storage
    → itinerary and segment modeling
    → configurable aggregation cubes
    → final analysis-ready market tables
```

The result is a reusable pipeline that turns raw survey records into processed tables suitable for traffic analysis, market diagnostics, competitive assessment, connectivity evaluation and strategic opportunity screening.
