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

## 4. Endorsement & Certification
To build trust in the analytics, the following labels are used in the workspace:
* **Promoted:** Applied to Gold tables that have passed all quality checks and the final `project_report` to indicate it as the "Single Source of Truth.".
