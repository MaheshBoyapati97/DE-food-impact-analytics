# DE-food-impact-analytics
A data engineering project comparing the environmental footprint and health profile of plant-based foods vs. conventional/animal-based foods, built on Microsoft Fabric using PySpark, OneLake, and Direct Lake Power BI.

# Plant-Based vs. Conventional Food Environmental & Health Impact Analytics

## Project Overview
I wanted a project that used real public data instead of generated/mock data, and that actually required dealing with messy source data rather than something pre-cleaned. Food seemed like a good fit — there's a well-known environmental impact study (Poore & Nemecek) and a large, genuinely messy open product database (Open Food Facts) that could be combined into one comparison platform.

## Objective
Build a working data engineering pipeline — raw ingestion, cleaning, transformation, and business-ready reporting — on Microsoft Fabric, using real public data sources.

## Business Problem
Consumers and researchers frequently want to compare the environmental and health tradeoffs of food choices, but the underlying data (product nutrition data, agricultural environmental impact data) sits in separate, differently structured sources. This project consolidates both so they can be compared directly, at the food-category level.

## Architecture Overview
The project follows a Medallion Architecture pattern:
- **Bronze**: Raw ingestion of both source datasets, landed as-received with no transformation logic applied.
- **Silver**: Cleaning, deduplication, null-handling, and category-mapping between the two sources.
- **Gold**: Business-ready fact tables joining health metrics to environmental footprint by food category.

---

## Phase 1: Data Sourcing & Fabric Environment Setup
Before building the pipeline, I identified the two data sources needed and set up the Fabric environment to build on.

### Data Domains & Strategy
| Source | Data Type | Refresh Frequency | Purpose |
| :--- | :--- | :--- | :--- |
| **Open Food Facts** | Product-level, raw | One-time bulk export (for this project) | Nutri-Score, NOVA group, nutrient panel, category tags |
| **Poore & Nemecek (2018) / Our World in Data** | Commodity-level reference data | Static (published dataset) | GHG emissions, land use, freshwater withdrawal per food category |

---

### Environment Setup Completed

### Key Updates:
* **Workspace Provisioning:** Created a dedicated Fabric workspace, `Plant_Vs_Conventional_Food_Analytics_Dev`, under Microsoft Fabric Trial Capacity.
* **Lakehouse Provisioning:** Created the primary Data Engineering storage container (Lakehouse), `lh_plant_vs_conventional_dev`, to hold the Bronze, Silver, and Gold layers.

### Technical Implementation Detail
- **Workspace:** `Plant_Vs_Conventional_Food_Analytics_Dev`
- **Lakehouse:** `lh_plant_vs_conventional_dev`

### Engineering Challenges & Resolutions
- Nothing to report yet at this stage — will document real issues as they come up in later phases.

---

## Phase 2: Raw Data Staging & Bronze Ingestion
*Not started yet.*

Plan: stage the Open Food Facts Parquet export and Poore & Nemecek CSVs in an ADLS Gen2 storage account, then create OneLake Shortcuts from the Lakehouse into that storage for zero-copy Bronze access — no data duplicated into Fabric itself.

---

## Phase 3: Silver Transformation
*Not started yet.*

Plan: deduplication and null-handling on Open Food Facts, unit standardization, and a category-mapping table linking branded products to Poore & Nemecek's ~40 commodity categories. This mapping is expected to be the hardest part — it isn't a clean 1:1 join.

---

## Phase 4: Gold Aggregation
*Not started yet.*

Plan: fact/dimension tables combining health metrics and environmental footprint by food category, structured for Power BI.

---

## Phase 5: Power BI Reporting (Direct Lake)
*Not started yet.*

Plan: a Direct Lake report comparing plant-based and conventional foods across both dimensions, with a basic scenario-modeling view (e.g., substitution impact).

---

## Tools and Technologies
- **Microsoft Fabric**: Lakehouse, OneLake, and Data Pipelines for the core platform.
- **PySpark**: Cleaning and transformation logic in Fabric Notebooks.
- **Delta Lake**: Underlying storage format for all medallion tables.
- **OneLake Zero-Copy Shortcuts**: Virtualized access to externally staged raw data.
- **Power BI (Direct Lake)**: Reporting layer.
- **Azure Data Lake Storage Gen2**: Raw data staging.

## Project Status
- [x] **Phase 1: Data Sourcing & Fabric Environment Setup**: Workspace and Lakehouse provisioned, data sources identified.
- [ ] **Phase 2: Raw Data Staging & Bronze Ingestion**
- [ ] **Phase 3: Silver Transformation**
- [ ] **Phase 4: Gold Aggregation**
- [ ] **Phase 5: Power BI Reporting (Direct Lake)**

## Data Attribution
- Product data © [Open Food Facts](https://world.openfoodfacts.org) contributors, ODbL license.
- Environmental impact data from Poore, J., & Nemecek, T. (2018). *Reducing food's environmental impacts through producers and consumers.* Science, 360(6392), 987–992, as republished by [Our World in Data](https://ourworldindata.org/environmental-impacts-of-food), CC BY.

## Author
Mahesh Boyapati
