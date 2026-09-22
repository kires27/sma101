# TASK1 — Build Guide

## Step 0 — Local DB foundation

- [x] Initialize `toolkit-stocks-v2` local DB. Create tables.
- [x] Start local DB as a **copy of production DB** (seed it from prod).
- Pick one SQLite file, WAL mode, single process.

## Step 1 — Raw data storage + parsers

- [ ] Expand local db and store as much useful data from raw files.
  - do i want to keep all the columns. do i want to switch up some rows?
  - compare each statement table with what yahoo finance provides.  
- [ ] Write a **bulk parser** that uploads raw data into the local DB.
  - raw data: `/mnt/kire/Games/home/edgar/`

## Step 2 — Non-statement data via yfinance

- [ ] Use **yfinance** to upload everything that is NOT a financial statement:
  - price, splits, dividends, and other niche data.
  - Applies to **all** stocks, including US.
- Keep this separate from the financial-statement parsers (Step 1).

## Step 3 — Priority + sync direction

- **New data (from pipelines) has higher priority** than production data.
- Sync order: insert pipeline data into local DB → *then* backfill from production DB.
- Production data arrives as **CSV**.
- **Conflicting data is ignored** (new data wins; do not overwrite).

## Step 4 — Pipeline dispatchers

- [ ] Build a mechanism that **differentiates which pipeline updates each stock**:
  - e.g. 5 stocks get an update — is each a US stock (EDGAR) or global (yfinance)?
- [ ] Build a mechanism that **selects which stocks need updating**:
  - **Financial data → fiscal calendar** (FYE per stock, poll in-window).
  - **Market data → daily** (recently-visited or all stocks, every ~30 min).
- Pipelines **run in parallel**. (aiohttp)

## Step 5 — Edgar pipeline (US stocks)

- Annual + quarterly financial statements.
- [ ] edgartools fetches → XBRL parse → store in local DB.

## Step 6 — yfinance pipeline (global stocks)

- Annual + quarterly financial statements.
- [ ] yfinance `.financials()` → normalize → store in local DB.

## Step 7 — Daily market pipeline

- Daily / ~30-min cadence.
- [ ] Uses yfinance to update **price** and niche data.
- Updates **all** stocks.
- May fill gaps edgar doesn't cover.
- **Must NOT mix with the yfinance financial-statement pipeline** (Step 6). Separate path.
- Split from finance pipeline by **flag**.

## Step 8 — Data flow: pipelines → local → production

- Pipelines write into **local DB**.
- From local DB, push to **production DB**.
- Production push respects priority (Step 3): new data wins, conflicts ignored.

## Step 9 — Error monitoring

- [ ] **Data logs** — structured log of what each pipeline did.
- [ ] **SQLite error table** — failures, retries, state.
- [ ] **Alerting** — email when pipelines fail / error rate high.
