# Retail Sales & Marketing Data Model (Power BI)

A Power BI project where I took messy, denormalized source data (raw exports, legacy CSV dumps, and spreadsheet extracts) and modeled it into a clean **star schema** — building out the dimensions, the fact layer, and a **junk dimension** for low-cardinality order flags — ready for reporting and analysis.

This project is focused on the **data modeling work itself**: profiling messy inputs, designing conformed dimensions, choosing the right grain for each fact table, and deciding what belongs in a junk dimension versus a standalone dimension.

## 📁 Repository Contents

| File | Description |
|---|---|
| `dataset.xlsx` | Raw source data — the "before" state. Multiple sheets simulating exports from different systems (orders, CRM, campaigns, inventory, finance), with intentional messiness (mixed keys, duplicate customer records, legacy references, free-text fields, wide/pivoted tables, etc.) |
| `project_datamodel.pbix` | The Power BI project — Power Query transformations, the modeled star schema, and report pages built on top of the clean model |

## 🎯 Project Goal

Real-world data rarely arrives ready for analysis. I set out to simulate that reality: raw data landing across inconsistent sheets with different grains, naming conventions, and structures — and then did the modeling work to turn it into something fast, intuitive, and correct to report from.

My workflow:
1. **Profiled the raw data** — identified grain, keys, duplicates, and inconsistencies in each source sheet
2. **Built conformed dimensions** — one clean, deduplicated table per business entity
3. **Built fact tables** at the right grain, linked to dimensions via surrogate/natural keys
4. **Consolidated scattered flag/attribute columns into a junk dimension**, instead of letting them bloat the fact table or become orphan dimensions
5. **Assembled the star schema** in Power BI and built report pages on top of it

## 🧩 Raw Source Data (`dataset.xlsx`)

The workbook mimics data pulled from several disconnected systems, including deliberately "messy" patterns to clean up:

- **`CUST_MASTER`** — customer master data, including `hash_key`/`source_id` columns typical of a change-data-capture or MDM extract
- **`ORDERS_2025`** / **`ORDERS_2026`** — order headers split across two annual sheets, with a legacy reference column, free-text `OrderNotes` and `GiftMessage`, and a numeric-coded `OrderChannel`
- **`order_line_items`** — order line-level detail (quantity, unit price, unit cost, discount %, line total)
- **`Address`**, **`cities`**, **`regions`** — normalized geography split across three separate tables
- **`products`**, **`subcategories`** — product master data, including an MDM-style `hash_key`/`source_id`, plus a `CategorySubcategory` column that packs two attributes into one delimited string (`electronics|phones`)
- **`CAMPAIGN_LOG`**, **`campaign_skus`** — marketing campaign headers/daily performance, and a comma-delimited list of promoted SKUs per campaign
- **`INVOICES`**, **`invoice_lines`**, **`payments`** — billing and collections data
- **`Sheet1`** / **`shipments`** — fulfillment/shipment tracking (duplicated sheet, another cleanup case)
- **`inventory`** — stock-on-hand pivoted wide, one column per month (`2025-01` … `2025-12`)
- **`sales_targets`** — monthly revenue targets by period
- **`exchange_rates`** — currency conversion rates by date
- **`customer_contacts`** — multiple contacts per customer, with an `IsPrimary` flag
- **`security`**, **`user_details`** — user-to-region mapping and account attributes, used for row-level security

## ⭐ Data Model (`project_datamodel.pbix`)

I transformed the raw sheets above in Power Query and modeled them into the following star schema:

### Dimensions
| Table | Purpose |
|---|---|
| `dim_customer` | One clean row per customer (built from `CUST_MASTER`, `customer_contacts`, and address/geography lookups) |
| `dim_product` | One clean row per product (built from `products`, with `CategorySubcategory` split into separate Category and Subcategory columns) |
| `dim_geo` | Conformed geography dimension (built from `Address`, `cities`, `regions`) |
| `dim_campaign` | Marketing campaign attributes (built from `CAMPAIGN_LOG`, `campaign_skus`) |
| `dim_date` | Standard calendar/date table with a Year–Quarter–Month hierarchy, used for time intelligence across every fact table |
| `dim_order_flags` | **Junk dimension** — consolidates low-cardinality order attributes (`Status`, `Priority`, `OrderChannel`, presence/absence of `GiftMessage`, etc.) into a single compact lookup table instead of letting them live as separate dimensions or bloat the fact table |

### Facts
| Table | Grain | Purpose |
|---|---|---|
| `fact_sales` | One row per order line | Core sales fact — quantities, prices, discounts, line totals |
| `fact_order_process` | One row per order | Order-level metrics and status/fulfillment tracking |
| `fact_campaign_spend` | One row per campaign per day | Daily marketing spend, impressions, and clicks |
| `fact_promotion_coverage` | Campaign ↔ product | Bridges which products were promoted in which campaigns |
| `fact_inventory` | One row per product per month | Unpivoted monthly stock-on-hand |
| `fact_sales_targets` | One row per period | Monthly revenue targets for actual-vs-target reporting |

### Other model objects
- **`_measures`** — a dedicated, empty table used purely to organize all measures (e.g. `total_sales`, `total_orders`) in one place, separate from any data table
- **`security`** — row-level security (RLS) table mapping users to regions, used to filter report data by the viewer's region

### Relationships
All fact tables connect to the shared dimensions (`dim_date`, `dim_geo`, `dim_customer`, `dim_product`, `dim_order_flags`) in a standard one-to-many star schema, avoiding snowflaking and keeping the model simple and performant.

## 📊 Reports

The `.pbix` includes report pages built on top of the model, including:
- Sales summary by year/quarter/month (using the `dim_date` hierarchy)
- KPI cards for total sales and total orders (via `_measures`)
- Regional breakdown by customer

## 🛠️ Tools Used

- **Power Query (M)** — extraction, cleaning, unpivoting, splitting delimited columns, deduplication
- **Power BI Desktop** — star-schema data modeling, relationships, row-level security

## 🚀 How to Use

1. Clone this repository
2. Open `project_datamodel.pbix` in [Power BI Desktop](https://powerbi.microsoft.com/desktop/)
3. Explore the **Model** view to see the star schema and relationships
4. Explore the **Power Query Editor** to see the transformations applied to each raw source table
5. `dataset.xlsx` is included so you can trace any modeled table back to its raw source

## 📌 Notes

- This is a portfolio project I built on synthetic data, designed to simulate common real-world data quality issues (split tables, inconsistent keys, wide/pivoted layouts, delimited multi-value fields, free text, legacy references) — and to demonstrate the dimensional modeling process end to end.
- Feel free to fork this repo and practice your own dimensional modeling on the same raw dataset.

---
*Feel free to adjust the sections above (screenshots, live report link, measure list, etc.) to match your final build before publishing.*
