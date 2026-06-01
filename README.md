# oytra-assessment
# Oytra Assessment — Automation & QA Developer

**Submitted by:** [Your Name]  
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
Full analysis in `Task1_QA_Report.pdf`.

---

## Task 2 — n8n API Integration Workflow

**File:** `Task2_Workflow_YourName.json`

### APIs Used
- **HackerNews API** (Primary) — `https://hacker-news.firebaseio.com/v2/topstories.json`
  - Free, no API key, no rate limits
  - Fetches top 5 story IDs
- **HackerNews Item API** (Enrichment) — `https://hacker-news.firebaseio.com/v2/item/{id}.json`
  - Fetches full details (title, score, comments) for each story

### How the Workflow Works
1. **Schedule Trigger** fires every 1 hour
2. **HTTP Request** fetches top 5 HackerNews story IDs
3. **Code Node** transforms the response — extracts and maps top 5 IDs
4. **HTTP Request (Enrich)** fetches full details for each story
5. **IF Node** checks if score > 100 → routes to HOT or WATCH branch
6. **Google Sheets** appends a digest row to "Morning Brief" sheet

### Error Handling
- All HTTP Request nodes have **"Continue On Fail"** enabled
- Failed calls are logged rather than crashing the workflow
- Error Trigger node catches any complete workflow failures

### Output
Results are written to Google Sheets with columns:
`Timestamp | Rank | Title | Score | Comments | Alert`

---

## Bonus Task — Uptime Monitor

**File:** `Bonus_UptimeMonitor_YourName.json`

### How It Works
1. **Schedule Trigger** pings `https://demo.realworld.show` every 5 minutes
2. **HTTP Request** hits the site with Continue On Fail enabled
3. **IF Node** checks if status code ≠ 200
4. **TRUE branch** → logs alert to Google Sheets "Alerts" tab
5. **FALSE branch** → logs healthy check to same sheet
6. **Error Trigger** → catches complete failures and logs them

### Columns logged
`Timestamp | URL | Status Code | Issue`

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
