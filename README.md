# End-to-End Earthquake Analytics on Microsoft Fabric

This repository contains an end-to-end data engineering and analytics project that processes global earthquake data. The platform automates data ingestion from a public API, implements a multi-layered Lakehouse architecture, builds a dimensional model for BI, and develops a machine learning model for tsunami prediction—all within the Microsoft Fabric ecosystem.

![Power BI Dashboard Screenshot](https://raw.githubusercontent.com/CarlosDiazData/Earthquake-Analytics/refs/heads/main/docs/powerbi_dashboard_preview.png)

---

## Table of Contents
1.  [Project Overview](#1-project-overview)
2.  [Key Features](#2-key-features)
3.  [Tech Stack](#3-tech-stack)
4.  [Data Architecture](#4-data-architecture)
5.  [Project Structure](#5-project-structure)
6.  [How to Run](#6-how-to-run)
7.  [Data Pipeline Explained](#7-data-pipeline-explained)

---

## 1. Project Overview

The goal of this project was to build a robust, automated analytics platform to analyze global earthquake events. The platform answers key questions such as:
*   What are the most seismically active regions in the world?
*   Are there temporal patterns in earthquake frequency or magnitude?
*   Can we predict the likelihood of a tsunami warning based on an earthquake's characteristics?

---

## 2. Key Features

*   **Automated Data Ingestion:** A daily pipeline orchestrates the ingestion of the latest earthquake data from the USGS public API.
*   **Medallion Architecture:** Data is structured into Bronze (raw), Silver (cleaned), and Gold (modeled) layers within a Fabric Lakehouse to ensure data quality and governance.
*   **Dimensional Modeling:** A Star Schema is implemented in the Gold layer with Fact and Dimension tables, optimized for fast analytical queries.
*   **Machine Learning:** A binary classification model (Random Forest) is trained and evaluated to predict tsunami warnings, including handling of imbalanced classes.
*   **Interactive BI Dashboards:** A Power BI report provides a global overview and temporal trend analysis.
---

## 3. Tech Stack

*   **Cloud Platform:** **Microsoft Fabric**
    *   **Data Storage & Processing:** Fabric Lakehouse (OneLake), Spark Notebooks (PySpark)
    *   **Orchestration:** Fabric Data Pipelines
    *   **Analytics & BI:** SQL Endpoint, Power BI (Semantic Models, Reporting)
*   **Languages:** **Python** (PySpark, Pandas), **SQL**
*   **Key Libraries:** `requests`, `pyspark.sql`, `pyspark.ml`, `matplotlib`
*   **Version Control:** **Git & GitHub**

---

## 4. Data Architecture

This project follows a modern Lakehouse architecture, leveraging the Medallion pattern for progressive data refinement.

**Data Flow:**
`USGS API` ➔ `Fabric Pipeline (triggers Notebook)` ➔ `Bronze Layer (Raw)` ➔ `Silver Layer (Cleaned)` ➔ `Gold Layer (Modeled)` ➔ `Power BI`

---

## 5. How to Run

1.  **Prerequisites:** An active Microsoft Fabric trial or subscription.
2.  **Setup Workspace:** Create a Fabric Workspace and a Lakehouse within it (e.g., `EarthquakeDataLakehouse`).
3.  **Upload Assets:** Upload the notebooks from the `notebooks/` directory to your Fabric Workspace and attach them to the Lakehouse.
4.  **Execute Notebooks:** Run the notebooks sequentially from `01` to `04`. The notebooks will create the Bronze, Silver, and Gold tables within your Lakehouse.
5.  **Create Pipeline:** Build a Data Pipeline in Fabric to orchestrate the execution of all notebooks and schedule it to run daily.
6.  **Build Power BI Report:**
    *   Create a new Power BI Semantic Model from your Lakehouse, selecting the tables in the `gold` schema.
    *   Verify the relationships between the fact and dimension tables.
    *   Build the report visuals.

## 6. Data Pipeline Explained

#### Bronze Layer: Raw Data Ingestion
*   The `01_ingest_earthquake_data.ipynb` notebook fetches data daily from the USGS GeoJSON API.
*   Data is saved in its raw, unaltered state as a Delta table (`bronze_usgs_earthquakes`) in the Lakehouse. This provides an immutable, auditable source of truth.

#### Silver Layer: Cleaned & Conformed Data
*   The `02_transform_earthquake_data.ipynb` notebook reads from the Bronze layer and performs critical data quality operations:
    *   **Type Casting:** Converts columns to appropriate data types (e.g., epoch milliseconds to timestamps).
    *   **Data Validation:** Filters out records with invalid data (e.g., impossible coordinates, negative depth).
    *   **Deduplication:** Removes duplicate events, keeping only the most recently updated record.
    *   **Enrichment:** Creates new features like `magnitude_category` and `depth_category`.
*   The result is saved as a clean, structured Delta table (`silver_earthquakes_cleaned`).

#### Gold Layer: Dimensional Model for Analytics
*   The `03_build_gold_layer.ipynb` notebook builds a **Star Schema** optimized for business intelligence:
    *   **Dimension Tables:** Creates `DimDate`, `DimLocation`, `DimMagnitude`, and `DimEventType` by extracting and deduplicating attributes from the Silver layer. Surrogate keys are generated to ensure efficient joins.
    *   **Fact Table:** Creates `FactEarthquakeEvents` by joining the Silver data with the dimension tables to look up the correct surrogate keys. This table contains numeric measures and foreign keys to the dimensions.
*   All tables are saved as Delta tables in the Lakehouse under a `gold` schema.

#### Machine Learning: Tsunami Prediction
*   The `04_ml_tsunami_prediction.ipynb` notebook builds a classification model:
    *   **Problem:** Predict if an earthquake will trigger a tsunami warning (a binary classification task).
    *   **Data Handling:** Addresses significant class imbalance by downsampling the majority class.
    *   **Model:** A `RandomForestClassifier` is trained using PySpark's MLlib.
    *   **Evaluation:** The model's performance is evaluated using robust metrics like **AUC-ROC** and **F1-Score**.
    *   **Output:** Predictions and probabilities are saved to a `silver_earthquake_ml_predictions` table for analysis in Power BI.

---



