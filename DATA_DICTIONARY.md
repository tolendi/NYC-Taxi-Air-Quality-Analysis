# Data Dictionary — Microsoft Fabric Learning Project

This document describes the data structure in the **Gold** layer (Warehouse/Lakehouse) for urban mobility, environmental, and economic analysis.

## 1. Table: gold_taxi_trips_daily
Contains aggregated metrics for NYC Yellow Taxi trips.

| Column Name | Data Type | Description | Transformation Logic |
| :--- | :--- | :--- | :--- |
| **pickup_date** | date | Date of the trips | Extracted from `tpep_pickup_datetime` |
| **pu_zone_id** | int | Taxi pickup zone ID | Original TLC Location ID |
| **trips_cnt** | bigint | Total daily trips | `COUNT(*)` |
| **total_revenue** | float | Total daily revenue | `SUM(total_amount)` |
| **avg_distance** | float | Average trip distance (miles) | `AVG(trip_distance)` |
| **avg_fare** | float | Average fare amount | `AVG(fare_amount)` |

## 2. Table: gold_air_quality_final_pivoted
Air quality data from OpenAQ API, pivoted by pollutant type and mapped to taxi zones.

| Column Name | Data Type | Description | Transformation Logic |
| :--- | :--- | :--- | :--- |
| **date** | date | Measurement date | Extracted from UTC timestamp |
| **taxi_zone_id** | int | NYC Taxi Zone ID | Joined via spatial/location mapping |
| **pm25** | float | Concentration of PM2.5 particles | Pivoted from `parameter` rows |
| **pm1** | float | Concentration of PM1 particles | Pivoted from `parameter` rows |
| **Borough** | varchar | NYC Borough name | Joined from `dim_zone` |

## 3. Table: dim_fx & dim_gdp
Macroeconomic indicators for financial context.

| Table | Column Name | Data Type | Description | Source |
| :--- | :--- | :--- | :--- | :--- |
| **dim_fx** | **avg_usd_eur_rate**| float | Daily/Avg USD to EUR rate | ECB API |
| **dim_gdp** | **gdp_usd** | float | Annual USA GDP in USD | World Bank API |
| **dim_gdp** | **year** | int | Calendar year | World Bank API |

## 4. Analytical Measures (DAX)
Calculated metrics and KPIs used in the Power BI dashboard for multi-domain analysis.

| Category | Measure Name | Data Type | Description | Logic / Formula |
| :--- | :--- | :--- | :--- | :--- |
| **Mobility** | **Total Revenue ($)** | Currency | Total gross revenue from taxi trips. | `SUM(total_revenue)` |
| **Mobility** | **Total Trips** | Integer | Total count of taxi rides. | `SUM(trips_cnt)` |
| **Mobility** | **Avg Distance** | Decimal | Average trip distance in miles. | `AVERAGE(avg_distance)` |
| **Mobility** | **Revenue_Airports** | Currency | Total revenue from airport pickup zones. | `SUM(gold_airports[total_revenue])` |
| **Environment**| **Avg PM2.5** | Decimal | Average concentration of PM2.5 particles. | `AVERAGE(pm25)` |
| **Environment**| **PM2.5 Filtered by Zone** | Decimal | PM2.5 levels mapped to specific Taxi Zones. | `CALCULATE(..., USERELATIONSHIP(...))` |
| **Environment**| **Avg UM003** | Decimal | Average count of ultra-fine particles (0.3µm). | `AVERAGE(um003)` |
| **Economy** | **Total Revenue EUR** | Currency | Revenue converted to EUR based on annual rates. | `SUMX(...) / LOOKUPVALUE(...)` |
| **Economy** | **GDP CAGR (2019-24)** | Percentage | 5-year compound annual growth rate of US GDP. | `((GDP2024/GDP2019)^(1/5))-1` |
| **Economy** | **GDP Index** | Decimal | Economic growth relative to 1975 baseline. | `DIVIDE(SUM(gdp_usd), BaseValue)` |

---

## Data Quality Rules (Business Logic)
1. **Taxi Filtering:** Records with `trip_distance` <= 0 or `fare_amount` < 0 are removed to eliminate telemetry errors.
2. **Missing Air Quality Data:** If hourly data is missing, the `avg_value` is calculated based on available data points for that day.
3. **Exchange Rates:** For weekends/holidays, a "Forward Fill" strategy is applied, using the rate from the last available business day.