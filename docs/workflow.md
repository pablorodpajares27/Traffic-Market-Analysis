# Workflow

This document shows the end-to-end workflow used in the BTS O&D Market Analysis project.

Unlike the architecture document, which explains the system design, this page focuses on the practical transformation path: how raw BTS survey files become final analysis-ready market tables.

---

## 1. Starting point: raw BTS O&D survey data

The process starts with raw BTS Origin & Destination survey files.

These files are not immediately suitable for market analysis because they are large, wide, period-based and difficult to interpret directly. They may contain many repeated fields for airports, carriers, coupons, distances and intermediate points.

A raw record can include information such as:

* reporting carrier;
* reporting year and month;
* number of coupons or segments;
* fare and tax amounts;
* passenger count;
* origin airport;
* intermediate airports;
* final destination;
* marketing and operating carriers;
* segment distances;
* purchase window group.

Example visual:

```md
![Raw BTS survey sample](../images/raw_table_sample.png)
```

The purpose of the workflow is to turn this kind of raw survey structure into tables that can be used for market, route and competition analysis.

---

## 2. Program 1 — Ingestion and standardization

The first program creates a clean storage layer from the raw files.

It performs the following steps:

1. scans the input folder for new BTS files;
2. detects the dataset type and reporting period;
3. archives the original input file;
4. extracts compressed files when needed;
5. identifies the useful tabular file;
6. converts the source file into Parquet;
7. stores the result in monthly or quarterly folders;
8. writes logs and processing metadata.

Example visual:

```md
![Program 1 ingestion](../images/program1_ingestion_console.png)
```

Typical output folders:

```text
Files by month/
Files by quarter/
Archive/
Logs/
```

At this stage, the project has moved from raw source files to standardized Parquet files that can be processed efficiently.

---

## 3. Program 2 — Itineraries and segments

The second program transforms the standardized Parquet files into two analytical datasets.

### Itineraries

The itinerary table keeps one row per complete O&D journey.

It answers questions such as:

* What market did the passenger travel?
* Was the journey nonstop or connecting?
* What was the fare?
* Which carrier appears first in the itinerary?
* How many segments were included?

### Segments

The segment table expands each itinerary into one row per coupon or segment.

It answers questions such as:

* Which airports were used in each leg?
* Which carriers operated each segment?
* How much distance belongs to each segment?
* Where do connections happen?

Example visual:

```md
![Program 2 itineraries and segments](../images/program2_itineraries_segments_console.png)
```

This stage is where the project moves from raw survey records into an analytical travel model.

---

## 4. Program 3 — Aggregated market cubes

The third program builds grouped analytical cubes from the itinerary and segment datasets.

The aggregation process is driven by YAML configuration files. These files define what to group by and which metrics to calculate.

Example aggregation outputs include:

* market by city pair and period;
* market by airport pair and period;
* market by carrier and period;
* market by itinerary type;
* market by number of segments;
* market by purchase window;
* market by WAC region.

Example visual:

```md
![Program 3 aggregation engine](../images/program3_aggregation_engine_console.png)
```

At this stage, the pipeline creates reusable market-level cubes such as:

```text
market_city_month
market_airport_month
market_first_marketing_carrier_month
market_itinerary_type_month
market_purchase_window_month
market_wac_quarter
```

These cubes are useful for detailed analysis, but they are still intermediate analytical building blocks.

---

## 5. Program 4 — Final analysis tables

The fourth program combines the aggregated cubes into final processed tables.

These tables are designed to be easier to interpret and use in market analysis than the individual cubes.

Example visual:

```md
![Program 4 analytics builder](../images/program4_analytics_builder_console.png)
```

Main outputs include:

| Final table                   | Main analytical purpose                                      |
| ----------------------------- | ------------------------------------------------------------ |
| `market_core_summary`         | Market size, revenue, fare, yield and trend analysis.        |
| `market_competition_summary`  | Carrier leadership, market share and concentration.          |
| `market_connectivity_summary` | Nonstop vs connecting mix and segment friction.              |
| `market_booking_summary`      | Purchase-window mix and booking behavior.                    |
| `market_airport_summary`      | Airport-pair drill-down and route-level detail.              |
| `market_wac_summary`          | Regional traffic flows and macro geographic view.            |
| `market_seasonality_summary`  | Quarterly seasonality and peak-period patterns.              |
| `market_opportunity_summary`  | Strategic opportunity scoring and market prioritization.     |
| `dim_period`                  | Period dimension for time filtering and reporting structure. |

---

## 6. From raw record to analysis-ready output

The transformation can be summarized as follows:

| Stage                 | Example output                                            | What changes                                             |
| --------------------- | --------------------------------------------------------- | -------------------------------------------------------- |
| Raw survey file       | DB1C / DB1B source file                                   | Original BTS structure, difficult to analyze directly.   |
| Standardized storage  | Monthly / quarterly Parquet                               | Periods, metadata and schemas are standardized.          |
| Itinerary model       | `itineraries`                                             | One row represents one complete O&D journey.             |
| Segment model         | `segments`                                                | One row represents one coupon or segment.                |
| Aggregated cubes      | `market_city_month`, `market_airport_month`, etc.         | Data is grouped by analytical dimensions.                |
| Final analysis tables | `market_core_summary`, `market_opportunity_summary`, etc. | Business-ready outputs are produced for market analysis. |

---

## 7. Example transformation

The repository includes simplified sample files showing the transformation path:

| Sample file                                      | What it represents                         |
| ------------------------------------------------ | ------------------------------------------ |
| `examples/raw_survey_sample.csv`                 | Simplified raw-style BTS survey records.   |
| `examples/itineraries_sample.csv`                | One row per complete O&D journey.          |
| `examples/segments_sample.csv`                   | One row per segment inside a journey.      |
| `examples/market_core_summary_sample.csv`        | Core market-period metrics.                |
| `examples/market_opportunity_summary_sample.csv` | Final strategic opportunity scoring layer. |

These examples are simplified and anonymized-style samples intended to show the structure of the pipeline outputs.

---

## 8. Final analysis table overview

The final layer of the project is summarized in the following visual reference.

```md
![Analysis tables overview](../images/analysis_tables_overview.png)
```

This overview shows how the final tables support different analytical use cases:

* core market diagnostics;
* competitive landscape analysis;
* connectivity and travel friction;
* booking behavior;
* airport-pair drill-down;
* regional flows;
* seasonality;
* route opportunity screening;
* period filtering and reporting structure.

---

## 9. Workflow summary

The full workflow can be summarized in one line:

```text
Raw BTS survey files
    → standardized Parquet files
    → itineraries and segments
    → aggregated market cubes
    → final analysis-ready tables
```

The result is a structured and reproducible workflow that turns raw O&D survey records into processed aviation market analysis tables.

