# oytra-assessment
# Oytra Assessment — Automation & QA Developer

**Submitted by:** Pratyush Kumar Saha 
**Role:** Associate Software Developer  
**Date:** June 2026

---

## Task 1 — Web App QA & Debug Report

**App tested:** [Conduit / RealWorld Demo](https://demo.realworld.show)  
**Method:** Manual exploratory testing across all major user flows — sign up, 
login, create/edit/delete article, profile, logout.

### Issues Found (6 total)

| # | Issue | Severity |
|---|---|---|
| 1 | No password strength validation on Sign-Up | High |
| 2 | Navbar broken on mobile view | Medium |
| 3 | Global Feed broken on mobile view | Medium |
| 4 | JWT token stored in localStorage (XSS risk) | High |
| 5 | No confirmation dialog before article delete | Medium |
| 6 | Cryptic error messages on login failure | Low |

### Root Cause Analysis
Issue #1 (Password Validation) was selected for deep root cause analysis.  
The app has no client-side or server-side password policy — allowing 
single-character passwords with no user feedback. Fix involves adding 
frontend validation + enforcing rules on the backend as a second layer.  
Full analysis in `Task1_QA_Report_PratyushKumarSaha.pdf`.

---

## Task 2 — n8n API Integration Workflow

**File:** `Task2_Workflow_PratyushKumarSaha.json`

### APIs Used
- **CoinGecko Markets API** (Primary) — `https://api.coingecko.com/api/v3/coins/markets?vs_currency=usd&order=market_cap_desc&per_page=5&page=1&sparkline=false`
  - Free, no API key required
  - Fetches top 5 cryptocurrencies by market cap including price and 24h change
- **CoinGecko Coin Detail API** (Enrichment) — `https://api.coingecko.com/api/v3/coins/{coinId}`
  - Fetches full details for the top ranked coin
  - Used to enrich the primary data with additional market information

### How the Workflow Works
1. **Schedule Trigger** fires every 1 hour
2. **HTTP Request** fetches top 5 cryptocurrencies from CoinGecko Markets API
3. **Code Node** transforms the response — extracts rank, name, symbol, 
   price, 24h change percentage, and coin ID
4. **HTTP Request (Enrich)** fetches full coin details for the #1 ranked coin
5. **IF Node** checks if 24h price change > 2% → routes to RISING or WATCH branch
6. **Google Sheets** appends a digest row to "Morning Brief" sheet

### Error Handling
- All HTTP Request nodes have **"Continue On Fail"** enabled
- Failed calls are logged rather than crashing the workflow
- Error Trigger node catches any complete workflow failures

### Output
Results are written to Google Sheets with columns:
`Timestamp | Rank | Coin | Price (USD) | 24h Change % | Alert`

### Google Sheet Link
`http://docs.google.com/spreadsheets/d/125tOo8FUMFXJM5eZrx4Dx-qQq9sO4m2k-Ro62xms0Jo/edit?gid=0#gid=0`

---

## Bonus Task — Uptime Monitor

**File:** `Bonus_UptimeMonitor_PratyushKumarSaha.json`

### How It Works
1. **Schedule Trigger** pings `https://demo.realworld.show` every 5 minutes
2. **HTTP Request** hits the site with Continue On Fail enabled
3. **IF Node** checks if status code ≠ 200
4. **TRUE branch** → logs alert to Google Sheets "Alerts" tab
5. **FALSE branch** → logs healthy check to same sheet
6. **Error Trigger** → catches complete failures and logs them

### Columns logged
`Timestamp | URL | Status Code | Issue`

### Google Sheet Link
`https://docs.google.com/spreadsheets/d/125tOo8FUMFXJM5eZrx4Dx-qQq9sO4m2k-Ro62xms0Jo/edit?gid=990008329#gid=990008329`

### Notable Finding
During testing, the uptime monitor detected that `demo.realworld.show` 
was not consistently returning 200 — consistent with the site instability 
documented in Task 1. The monitor correctly routed to the alert branch.

---

## Tools & Technologies Used
- **n8n** (self-hosted via npx) — workflow automation
- **HackerNews Firebase API** — free public data source
- **Google Sheets API** — output/notification channel
- **Chrome DevTools** — for QA testing and security analysis
