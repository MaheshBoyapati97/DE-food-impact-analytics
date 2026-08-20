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

![Storage_Account_Overview](screenshots/Storage%20Account%20Overview.png)

Two containers, named for what's actually in them rather than where they came from — `food-nutrition-health-data` and `food-environmental-impact` — since I figured six months from now I won't remember what "owid" stands for.

![Containers_List](screenshots/Containers%20List.png)

Grabbed the three Poore & Nemecek CSVs straight from Our World in Data's grapher URLs. Open Food Facts was the bigger job — pulled the Parquet export (~7.2GB) from Hugging Face through Azure Cloud Shell instead of downloading to my laptop first, so the transfer happened inside Azure's network rather than over my home connection twice. Before uploading it anywhere, read it back with pandas to make sure it wasn't truncated — came back with 4,655,195 rows, so the file was intact.

![Food_Parquet_Blob_Details](screenshots/Food%20Parquet%20Blob%20Details.png)

Then created two OneLake Shortcuts (ADLS Gen2, SAS token auth) from the Lakehouse into those two containers. Skipped the "Transform to Delta" option Fabric offers during shortcut setup — didn't want any transformation logic sneaking into Bronze before I've even written the Silver notebook.

![Fabric_Lakehouse_Shortcuts](screenshots/Fabric%20Lakehouse%20Shortcuts.png)

**Technical detail:**
- Storage account: `stfoodimpactdev`
- Containers: `food-nutrition-health-data`, `food-environmental-impact`
- Shortcuts: same names, under Lakehouse `Files/`
- Files: `food.parquet` (~7.2GB), `ghg-per-kg-poore.csv`, `land-use-per-kg-poore.csv`, `water-withdrawals-per-kg-poore.csv`

**What actually went wrong:**

Spent a good chunk of time on a completely unrelated problem first — my Azure for Students subscription got disabled, all credit gone, and I hadn't run anything that should've cost real money. Turned out an old Databricks project (a different repo) had left a NAT Gateway and VNet running in the background for months, billing hourly regardless of whether I was actually using it. Cost had nothing to do with how often I ran a pipeline — it was just sitting there. Deleted that resource group, upgraded to Pay-As-You-Go, set a $5 budget alert so this doesn't happen silently again.

Then, once I got to actually uploading: `az storage blob upload --auth-mode login` just hung with no error. Turns out being subscription Owner isn't the same as having the `Storage Blob Data Contributor` role — that's a separate, data-plane-level permission the CLI needs and doesn't tell you about clearly. Worked around it with an account-key-based SAS token from the portal instead.

And then a dumb one that cost me twenty minutes: pasted a SAS token into a bash variable without quotes. SAS tokens are full of `&` characters, and bash reads unquoted `&` as "run this in the background" — so it silently split my token into like seven separate background jobs and only kept the first fragment. Got `403 AuthorizationPermissionMismatch` and assumed it was a permissions problem again, when it was actually just a quoting bug. Lesson: always wrap tokens in single quotes.

**Verifying the shortcuts actually work, not just exist:**

​```python
# Open Food Facts shortcut
df_food = spark.read.parquet("Files/food-nutrition-health-data/food.parquet")
df_food.printSchema()
print(df_food.count())

# OWID shortcut
df_env = spark.read.csv("Files/food-environmental-impact/ghg-per-kg-poore.csv", header=True)
df_env.show(5)
​```

![Notebook_Shortcut_Verification](screenshots/Notebook%20Shortcut%20Verification.png)

---

## Phase 3: Silver Transformation
Not started.

This is where the real cleaning happens — deduping Open Food Facts, handling nulls, standardizing units, and building the category-mapping table that links thousands of branded products to ~40 commodity categories from the Poore & Nemecek side. That mapping isn't a clean join and I expect it to take a while.

## Phase 4: Gold Aggregation
Not started. Fact/dimension tables combining health and environmental metrics by food category.

## Phase 5: Power BI Reporting
Not started. Direct Lake report comparing plant-based vs. conventional, maybe a basic substitution-scenario view if there's time.

---

## Stack
Microsoft Fabric (Lakehouse, OneLake, Data Pipelines), PySpark, Delta Lake, Power BI (Direct Lake), Azure Data Lake Storage Gen2, Azure Cloud Shell / AzCopy.

## Status
- [x] Phase 1: Fabric Environment Setup
- [x] Phase 2: Raw Data Staging & Bronze Ingestion via OneLake Shortcuts
- [ ] Phase 3: Silver Transformation
- [ ] Phase 4: Gold Aggregation
- [ ] Phase 5: Power BI Reporting

## Data credit
- Open Food Facts data © Open Food Facts contributors, ODbL license.
- Environmental data: Poore, J., & Nemecek, T. (2018). *Reducing food's environmental impacts through producers and consumers.* Science, 360(6392), 987–992. Via Our World in Data, CC BY.

## Author
Mahesh Boyapati
