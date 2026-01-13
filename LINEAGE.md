# Data Lineage Diagram

This diagram represents the end-to-end data flow from raw external sources to the final Power BI reporting layer using the Medallion Architecture.

## 1. Data Ingestion (Bronze Layer)
* **Sources**: NYC TLC (Parquet), OpenAQ (API), World Bank (API), ECB (CSV).
* **Ingestion Tools**: Data Factory Pipelines and Notebooks.
* **Storage**: Data is landed in `LH_Data` (Files/Bronze).

## 2. Transformation (Silver Layer)
* **Processing**: PySpark Notebooks (`air_silver`, `silver_taxi_trips`).
* **Logic**: Schema enforcement, data cleaning, and filtering (e.g., removing zero-distance trips).
* **Storage**: Tables are stored in Delta format in `LH_Data` (Tables/Silver).

## 3. Serving & Analytics (Gold Layer)
* **Processing**: Notebooks (`air_gold`, `gold_fact`) and SQL Analytics Endpoint.
* **Logic**: Aggregations, PIVOT operations for air quality, and Star Schema modeling.
* **Storage**: Final analytics tables (e.g., `gold_taxi_trips_daily`, `dim_fx`).

## 4. Visualization
* **Report**: `project_report` connected via **DirectLake** mode for real-time performance.