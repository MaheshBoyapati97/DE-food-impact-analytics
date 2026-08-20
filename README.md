# DE-food-impact-analytics
A data engineering project comparing the environmental footprint and health profile of plant-based foods vs. conventional/animal-based foods, built on Microsoft Fabric using PySpark, OneLake, and Direct Lake Power BI.

# Plant-Based vs. Conventional Food Environmental & Health Impact Analytics

I wanted a project that used real public data instead of generated/mock data, and that forced me to deal with actually messy source data instead of something pre-cleaned. Food turned out to be a good fit for that — there's a well-known environmental impact study (Poore & Nemecek) and a huge, genuinely messy open product database (Open Food Facts), and combining the two lets you compare plant-based vs. conventional foods on both environmental and health grounds at once.

The goal is a working pipeline end to end — raw ingestion, cleaning, transformation, reporting — on Microsoft Fabric, built the way a real team would build it rather than a tutorial version.

## Architecture

Standard medallion setup on one Fabric Lakehouse:
- **Bronze** — raw data landed via OneLake Shortcuts, untouched
- **Silver** — cleaned Open Food Facts data, plus a mapping table linking it to the Poore & Nemecek commodity categories
- **Gold** — the actual comparison tables, feeding a Power BI report over Direct Lake

## Data

| Source | What it gives me |
|---|---|
| Open Food Facts | Nutri-Score, NOVA group, nutrients, category tags — messy, raw, needs real cleaning |
| Poore & Nemecek (2018), via Our World in Data | GHG emissions, land use, freshwater withdrawal per kg, across ~40 food categories — curated research data, treated as reference data rather than something to "clean" |

---

## Phase 1: Fabric Environment Setup

Set up the Fabric side before touching any data:
- Fabric workspace: `Plant_Vs_Conventional_Food_Analytics_Dev` (Trial Capacity, Central US)
- Lakehouse: `lh_plant_vs_conventional_dev`

Nothing to report on the challenges front for this one — it was just clicking through the Fabric UI.

---

## Phase 2: Raw Data Staging & Bronze Ingestion via OneLake Shortcuts

This phase ended up being most of the actual work so far, and most of the real problems.

Set up an ADLS Gen2 storage account (`stfoodimpactdev`, Central US, Standard/LRS, hierarchical namespace on) to stage both datasets before bringing them into Fabric.

![Storage Account Overview](screenshots/Storage_Account_Overview.png)

Two containers, named for what's actually in them rather than where they came from — `food-nutrition-health-data` and `food-environmental-impact` — since I figured six months from now I won't remember what "owid" stands for.

![Containers List](screenshots/Containers_List.png)

Grabbed the three Poore & Nemecek CSVs straight from Our World in Data's grapher URLs. Open Food Facts was the bigger job — pulled the Parquet export (~7.2GB) from Hugging Face through Azure Cloud Shell instead of downloading to my laptop first, so the transfer happened inside Azure's network rather than over my home connection twice. Before uploading it anywhere, read it back with pandas to make sure it wasn't truncated — came back with 4,655,195 rows, so the file was intact.

![Food Parquet Blob Details](screenshots/Food_Parquet_Blob_Details.png)

Then created two OneLake Shortcuts (ADLS Gen2, SAS token auth) from the Lakehouse into those two containers. Skipped the "Transform to Delta" option Fabric offers during shortcut setup — didn't want any transformation logic sneaking into Bronze before I've even written the Silver notebook.

![Fabric Lakehouse Shortcuts](screenshots/Fabric_Lakehouse_Shortcuts.png)

**Technical detail:**
- Storage account: `stfoodimpactdev`
- Containers: `food-nutrition-health-data`, `food-environmental-impact`
- Shortcuts: same names, under Lakehouse `Files/`
- Files: `food.parquet` (~7.2GB), `ghg-per-kg-poore.csv`, `land-use-per-kg-poore.csv`, `water-withdrawals-per-kg-poore.csv`

**What actually went wrong:**

Spent a good chunk of time on a completely unrelated problem first — an old Databricks project (a different repo) had left a NAT Gateway and VNet running in the background for months, billing hourly regardless of whether it was actually being used. Cost had nothing to do with how often a pipeline was run — it was just sitting there. Deleted that resource group, upgraded the subscription, set a $5 budget alert so this doesn't happen silently again.

Then, once I got to actually uploading: `az storage blob upload --auth-mode login` just hung with no error. Turns out being subscription Owner isn't the same
