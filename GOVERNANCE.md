# Data Governance & Quality Policies

This section outlines the rules and standards applied to ensure data reliability, security, and traceability within the Microsoft Fabric environment.

## 1. Data Quality Standards (Medallion Rules)
To maintain high-quality insights, the following validation rules are implemented:
* **Completeness:** Records with missing critical identifiers (e.g., `LocationID` or `Date`) are quarantined or dropped during the Silver transformation.
* **Integrity:** Taxi trips with `trip_distance` <= 0 or `total_amount` < 0 are filtered out as telemetry errors.
* **Consistency:** All currency values are standardized to USD in the Silver layer, with an optional conversion to EUR in the Gold layer using the `dim_fx` reference table.

## 2. Security & Access Control
Data access is managed through Microsoft Fabric's integrated security model:
* **Workspace Roles:** Access is restricted using Role-Based Access Control (RBAC), dividing users into Admins, Members, and Viewers.
* **Compute Isolation:** Transformation logic is isolated within Spark Notebooks, ensuring that raw data (Bronze) is not directly accessible by end-report users.

## 3. Data Lineage & Traceability
* **Automatic Lineage:** Microsoft Fabric's Lineage View is used to track data provenance from API sources to Power BI dashboards.
* **Impact Analysis:** Before modifying any upstream Notebook (e.g., `air_silver`), the Lineage View is used to perform impact analysis on downstream Gold tables.

## 4. Endorsement & Certification (Planned)
*Note: Endorsement features are subject to tenant administration policies in the current environment.*

In a production environment, the following labels will be applied to maintain the "Single Source of Truth":
* **Promoted Status:** Will be assigned to all Gold-layer tables (e.g., `gold_taxi_trips_daily`) once the automated DQ (Data Quality) tests in the Spark Notebooks pass.
* **Certified Status:** The final `project_report` will be certified by the Data Governance team to ensure the metrics (e.g., Total Revenue, Avg PM2.5) are official and audited.

