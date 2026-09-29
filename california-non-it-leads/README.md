# California Non-IT Leads

n8n workflow that discovers local (non-IT) business leads across California. It builds randomized place-search queries, queries Serper.dev's Google Places API, filters out tech/software companies, crawls each business website to extract a contact email, and appends the cleaned leads to a Google Sheet.

- **Workflow name:** California Non-IT Leads
- **File:** `California Non-IT Leads.json`
- **Status:** Active
- **Execution order:** v1

---

## How It Works

1. A **schedule trigger** starts the run.
2. `Define Search Matrix` builds a matrix of ~230 business categories across 10 California regions, shuffles it, and returns 5 random queries per execution (roughly ~100 businesses per run).
3. Each query is sent to **Serper.dev `/places`** to retrieve local businesses.
4. Results are parsed and filtered against a **non-IT blacklist** (software, tech, cloud, IT services, web design, developer, digital agency, cybersecurity) in two passes.
5. Leads are processed in a batch loop, one at a time.
6. Each business website is crawled and a **contact email** is regex-extracted from the HTML.
7. The cleaned lead is appended to a **Google Sheets** tab.

## Node Flow

```
Schedule Trigger
  -> Define Search Matrix (Code)
  -> Serper.dev Search (HTTP POST)
  -> Parse Business Records (Code)
  -> Non-IT Blacklist & Classifier (Code)
  -> Batch Processor(prevents server) (Split In Batches)
       -> Crawl Business Website (HTTP GET)
       -> Regex Email & Data Extractor (Code)
       -> Save to Google Sheets (append)
       -> Batch Processor(prevents server)   (loop)
```

---

## Nodes

| Node | Type | Purpose |
|------|------|---------|
| `Schedule Trigger` | Schedule Trigger | Starts runs on an interval |
| `Define Search Matrix` | Code | Builds categories × CA regions, returns 5 random queries |
| `Serper.dev Search` | HTTP Request | Calls Serper.dev Google Places API |
| `Parse Business Records` | Code | Maps places to leads, first IT blacklist pass |
| `Non-IT Blacklist & Classifier` | Code | Second IT blacklist pass |
| `Batch Processor(prevents server)` | Split In Batches | Processes leads one at a time |
| `Crawl Business Website` | HTTP Request | Fetches each business homepage |
| `Regex Email & Data Extractor` | Code | Extracts the first valid email from HTML |
| `Save to Google Sheets` | Google Sheets | Appends the lead to the sheet |

---

## Search Coverage

- **Categories:** ~230 local service categories (contractors, clinics, restaurants, salons, legal, finance, manufacturing, and more).
- **Regions:** California, Los Angeles County, Orange County, San Diego County, San Francisco Bay Area, Sacramento Metro, Inland Empire, Central Valley, Silicon Valley, Central Coast.
- **Per run:** 5 random category × region queries.

## Output Fields

| Field | Source |
|-------|--------|
| `Business Type` | Place category |
| `name` | Business name |
| `phone number` | Place phone (or `N/A`) |
| `address` | Place address |
| `website` | Place website (or `N/A`) |
| `email` | First valid email found on the site (or `N/A`) |

---

## Google Sheet

- **Document:** `California Non-IT Leads`
- **Sheet / tab:** `Sheet1`
- Leads are **appended** using auto-mapped input data.

## Credentials & Services Required

| Credential | Used By |
|------------|---------|
| Google Sheets OAuth2 | Save to Google Sheets |
| Serper.dev API key | Serper.dev Search (sent as `X-API-KEY` header) |

---

## Security & Setup Notes

Sensitive values were redacted from this public copy. Before running, replace them with your own:

- `YOUR_GOOGLE_SHEET_ID` — your leads spreadsheet ID
- `YOUR_SERPER_API_KEY` — your Serper.dev API key (in the `Serper.dev Search` header)
- `REPLACE_WITH_CREDENTIAL_ID` / `REPLACE_WITH_CREDENTIAL_NAME` — attach your own n8n credentials

Consider moving the Serper API key into an **HTTP Header Auth** credential rather than storing it in the node URL/headers.

Setup steps:

1. Import the JSON into n8n.
2. Attach a Google Sheets credential and point the node at your spreadsheet/tab.
3. Add your Serper.dev API key to the `Serper.dev Search` node.
4. Adjust the batch size / schedule as needed and activate the workflow.
