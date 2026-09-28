# Handoff: Cedar Rapids Supply Shed — S1 vs S2 Supply-Chain Scenarios

*Last updated: 2026-09-26*

## 1. What this is

An R + Quarto analysis comparing two agricultural supply-chain scenarios for a **50-mile supply shed around Cedar Rapids, Iowa**, traced from field to **first buyer** (with extensions beyond it):

- **S1 (current):** corn and soybeans on today's corn/soy land (USDA Cropland Data Layer 2025).
- **S2 (hypothetical):** the same 3.24M acres reallocated to 10 commodities: corn, soybeans, oats, wheat, barley, alfalfa hay, apples, cattle (cow-calf), sheep (wool only) and goats (meat).

**Deliverable:** `supplyshed_scenarios.html` is a single self-contained file (about 11 MB). Only the basemap tiles need internet. Its source is `supplyshed_scenarios.qmd`.

## 2. Current results (as rendered)

| Metric | S1 | S2 |
|---|---|---|
| Farm-gate value | $2.52B | $2.17B |
| GHG, farm + haul to first buyer | 857 kt CO₂e | 897 kt CO₂e |
| First buyers used / companies | 52 / 49 | 67 / 63 |
| Buyer concentration (HHI, shed value) | 532 | 510 |
| Value retained in shed (direct) | $1.02B (41%), range $0.81–1.30B | $0.66B (30%), range $0.42–0.96B |
| Sales to locally owned first buyers | 57% | 73% |
| Median farm (584 crop ac) net return after $274/ac land charge | −$27k | −$180k |
| Median farm return to land & management | +$133k | −$20k |
| Median farm income swing (SD, 2006–2025 replay) | $93k | $53k |
| Expected annual insured weather loss | $70M (2.8%) | $106M (5.7%); crops only $62M (4.0%) |
| Weather-loss swing (SD) / worst year | $87M / $306M (2012) | $48M / $217M (2012) |
| Sales to buyers within 250 m of a FEMA 100-yr flood zone | 60% | 52% |

**Headline story:** S2 has lower output and retains less value in total, but it is **steadier** (income and weather losses swing less) and sells more to **locally owned** buyers. Its new markets are **thin**: wool has one cooperative, and goats and hay go mainly to Kalona Sales Barn.

## 3. Report sections (in order)

1. Scenarios at a glance: circle diagram, area proportional to output; the soybean icon is a custom SVG.
2. Field to consumer: a 10-step supply chain with a data source per step (`data/params/supply_chain_steps.csv`).
3. Summary: KPI cards and a total economic output card.
4. Maps: 5 km hexagons showing value and GHG per acre, flows to first buyers, and buyer sites.
5. Economic value: output by commodity, faceted and grouped.
6. GHG emissions: by source and by commodity.
7. Buyer concentration: top companies and HHI by commodity.
8. Producer-level comparison: representative farms (small, median, large), income-risk replay, net return by enterprise, cost table, and a hexagon change map.
9. Economic value retained in the shed: leakage by cost category and first-buyer ownership.
10. Climate risk: RMA weather loss costs, FEMA flood exposure, and NOAA storm frequency.
11. Cargill & ADM: field to first buyer (flow map, Sankeys drawn to the same pixels-per-ton scale).
12. Level-2 buyers (Tradeverifyd): customers of first-buyer companies.
13. U.S. export destinations (Tradeverifyd / UN Comtrade).
14. Corporate ownership of first buyers (Tradeverifyd affiliates).
15. Data tables, then Methods & assumptions: data sources table (`data/params/data_sources.csv`, all links checked Sep 2026) and limitations.

## 4. Repository layout

```
supplyshed_scenarios.qmd / .html   Report source / rendered output
_quarto.yml                         Quarto project (renders the .qmd only)
run_all.R                           Rebuild script (see gaps in §6)
R/                                  Pipeline scripts (numbered by step)
data/params/                        EDITABLE assumptions (CSV); change these, then rebuild
data/derived/                       Model outputs read by the report (.rds, .gpkg)
data/raw/                           Downloads (~400 MB; re-downloadable, see §6)
```

### Pipeline scripts

| Script | Does | Main output |
|---|---|---|
| `00_shed.R` | 50-mi buffer around Cedar Rapids (EPSG:5070) | `shed.gpkg`, `center.gpkg` |
| `01_counties.R` | 22 intersecting counties + share in shed | `shed_counties.gpkg` |
| `02_nass.R` | NASS county/state yields and prices | `nass.rds` |
| `03a_warehouses.R` | Scrapes IDALS licensed grain warehouses | `data/raw/ia_licensed_grain_warehouses.csv` |
| `03_buyers.R` | Curated buyers + elevators, geocoded (Census, then OSM) | `buyers.gpkg` (67 buyers) |
| `04_hex.R` | CDL pixel counts → 5 km hexagons | `hex.gpkg` (1,005 hexes) |
| `05_model.R` | **Core model:** land, production, value, GHG, Huff allocation to buyers, HHI | `results.rds` |
| `06a/b_tv_*.py`, `06c_level2.R` | Tradeverifyd entity resolution; level-2 buyers | `level2.rds` |
| `07a/b_tv_*.py`, `07c/d_*.R` | Tradeverifyd trade flows; affiliates | `tradeflow.rds`, `affiliates.rds` |
| `08a_farmsize.R` | Census of Ag 2022 acre-weighted farm sizes | `farm_size_classes.rds` |
| `08b_history.R` | NASS 2006–2025 yields and prices | `history.rds` |
| `08c_farms.R` | Representative farms, risk replay, hexagon change metrics | `farms.rds` |
| `09_retention.R` | Value retained in shed and buyer ownership | `retention.rds` |
| `10a_rma.R` | RMA cause-of-loss + summary of business → loss costs | `rma_shed.rds` |
| `10b_nfhl.py` | FEMA flood zones (paged REST download) | `data/raw/fema/*.geojson` |
| `10c_exposure.R` | Flood exposure (pixels, buyers), NOAA storms | `exposure.rds` |
| `10d_climate.R` | Scenario climate-risk metrics | `climate.rds` |
| `tv_client.py` | Minimal JSON-RPC client for the Tradeverifyd MCP endpoint | — |

## 5. Key assumptions (all editable in `data/params/`)

- **`commodities.csv`:** S2 land shares (corn 30%, soy 20%, cattle 14%, alfalfa 10%, oats 8%, wheat 6%, barley 4%, sheep 4%, goats 3%, apples 1%); N rates; stocking (cattle 0.5 cows/ac, sheep 0.94 ewes/ac, goats 0.96 does/ac); cattle value = ERS 2025 Heartland $1,227.54/cow. `commodities_v1_backup.csv` holds the earlier, superseded stocking and cattle settings.
- **`factors.csv`:** IPCC 2006 Tier 1 N₂O and CH₄ factors; AR6 GWPs (CH₄ 27, N₂O 273); EPA 2025 truck factor 0.186 kg CO₂/short ton-mile; road circuity 1.3; Huff distance-decay exponent 2.
- **`farm_costs.csv`:** non-land costs (ISU 2026 A1-20, USDA ERS 2025, Univ. of Missouri G686/G691/G712). A uniform land charge of $274/ac and a cropped share of 85% are set in `08c_farms.R`.
- **`local_shares.csv`:** share of each cost category spent inside the shed, with low and high values. These are **analyst assumptions**, except land (25% non-resident ownership, ISU 2022 survey).
- **`buyer_ownership.csv`:** buyer headquarters and ownership class. Unlisted state-licensed elevators are assumed locally owned.
- **`tv_location_overrides.csv`:** manual country corrections for Tradeverifyd records.
- **Buyer attractiveness** in the Huff model: processors 10, nearby ethanol plants 6, elevators and auctions 1, Kalona 3 (set in `data/raw/buyers_curated.csv`).

## 6. Setup and rebuild

**Requirements:** R 4.6+ (sf, terra, exactextractr, tigris, leaflet, plotly, DT, rnassqs, tidygeocoder, rvest, readxl, pdftools, countrycode, data.table), Quarto 1.10+, Python 3 (standard library only).

**Secrets (not in this repository; keep it that way):**
- `.Renviron` in the project root with `NASSQS_TOKEN=<your USDA NASS Quick Stats key>`.
- `.mcp.json` in the project root with the Tradeverifyd MCP server URL and bearer token (read by `R/tv_client.py`).
- Both were left in `~/Downloads/supplychains/` when the project moved. Copy them into the project root to rebuild, and **add both to `.gitignore`**.

**Quick re-render after editing params:** `Rscript R/05_model.R && Rscript R/08c_farms.R && Rscript R/09_retention.R && Rscript R/10d_climate.R && quarto render supplyshed_scenarios.qmd`

**`run_all.R` gaps:** these raw downloads were done by one-off commands and still need to be scripted:
- CDL 2025 clip: CropScape `GetCDLFile?year=2025&bbox=276000,2036000,437100,2197100`, saved to `data/raw/CDL_2025_shed.tif`.
- RMA `colsom_YYYY.zip` and `sobcov_YYYY.zip` for 2006–2025 from `pubfs-rma.fpac.usda.gov/pub/Web_Data_Files/Summary_of_Business/`, saved to `data/raw/rma/`.
- NOAA `StormEvents_details-ftp_v1.0_dYYYY_c*.csv.gz` for 2006–2025, saved to `data/raw/noaa/`.
- ERS cost-and-returns CSVs (corn, soybeans, oats, wheat, barley, cow-calf), saved to `data/raw/ers/`.
- EPA EF Hub 2025 xlsx; AMS 2153 goat PDF; ISU A1-20 PDF (download via R, since curl gets a 403); Missouri budgets, saved to `data/raw/budgets/`.
- `R/03_buyers.R` expects `data/raw/buyers_curated.csv` (present).
- `R/07b_tv_affiliates.py` expects `data/derived/buyers_list.csv` (written in `run_all.R`).

## 7. Known limitations and caveats

- **Tradeverifyd data is noisy.** Records are company-level, not plant-level. Cargill is split across many entities and under-counted. Trade flows cover **2023 only and 4 partners** (China, Mexico, Japan, South Korea) with no tonnages. The affiliate link direction is unreliable.
- **Farm economics use economic costs** (they include operator labor and ownership costs). Net returns are negative in both scenarios at 2025 prices. Apples are at full production at the national commodity price, and sheep earn wool only. There are no economies of scale.
- **Retention is direct only.** It has no multipliers (IMPLAN would add them) and excludes processor wages and buyer margins. The local shares are assumptions.
- **Climate:** only corn and soy use shed-level RMA data. Minor crops use Iowa statewide data, barley borrows oats' rates, livestock uses the pasture rainfall-index product, and apples are not covered. Flood status is approximate for buyers placed at town centroids.
- **Elevators** come only from the state-licensed list; large federally licensed co-ops are under-represented.
- S2 shares and yields for wheat, barley and apples (U.S. averages) are illustrative.

## 8. Open decisions and next steps

- **GitHub:** the repository is `github.com/adampdixon/TRACEUNSEEN_Hackathon` (appears private). Nothing except `.gitattributes` is committed yet. Recommended `.gitignore`: `.Renviron`, `.mcp.json`, `.DS_Store`, `.quarto/`, `data/raw/` (or at least `rma/`, `noaa/`, `budgets/`, `isu_*.pdf`).
- **Before going public:** check Tradeverifyd's terms of use for redistributing its data (raw JSON, derived `.rds` files, and the tables in the HTML). Don't commit copyrighted extension PDFs.
- **Hosting:** GitHub Pages can serve the HTML (rename it to `index.html` or link to it).
- **Possible extensions discussed:**
  - Forward-looking climate: gridMET weather sensitivity with LOCA2 projections.
  - Transition risk: carbon prices and nitrogen price exposure.
  - A cash-cost return view and adjustable prices for the farm analysis.
  - Crop Sequence Boundaries (field-level) instead of hexagons.
  - IMPLAN multipliers.
  - Third-level Tradeverifyd buyers.
