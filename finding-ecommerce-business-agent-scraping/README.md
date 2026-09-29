# Finding Ecommerce Business through Agent (Scraping)

n8n workflow that finds ecommerce businesses in Lahore using Apify's Google Places crawler, waits for the scrape to finish, pulls the dataset, keeps only poorly-rated businesses with negative reviews, removes duplicates, and writes them to a Google Sheet.

- **Workflow name:** Finding Ecommerce Business through Agent(Scraping)
- **File:** `Finding Ecommerce Business through Agent(Scraping).json`
- **Status:** Inactive (manual run)
- **Execution order:** v1

---

## How It Works

1. The workflow is started **manually**.
2. It launches an **Apify** actor run (`compass~crawler-google-places`) with a large list of ecommerce-related search strings scoped to Lahore.
3. It waits 60 seconds, then polls the run status until `status === SUCCEEDED`.
4. Once complete, it downloads the run's dataset items as JSON.
5. Businesses are filtered to those with **rating < 3.5** and at least one review of **2 stars or less** (opportunities for outreach).
6. Results are **deduplicated by `place_id`**.
7. Leads are appended/updated in a **Google Sheets** tab.

## Node Flow

```
Manual Trigger
  -> Apify (launch actor run)
  -> Wait (60 seconds)
  -> Check Run Status
  -> If it is done
       ├─ false -> Wait (poll again)
       └─ true  -> Datasets Items
                -> Filter by Rating & Negative Review
                -> Dedupe by place_id
                -> Append or update row in sheet
```

---

## Nodes

| Node | Type | Purpose |
|------|------|---------|
| `When clicking 'Execute workflow'` | Manual Trigger | Starts the workflow manually |
| `Apify` | HTTP Request | Launches the Apify Google Places crawler run |
| `Wait` | Wait | 60 second pause before checking status |
| `Check Run Status` | HTTP Request | Polls the Apify run status |
| `If it is done` | IF | Loops until the run succeeds |
| `Datasets Items` | HTTP Request | Downloads the dataset items |
| `Filter by Rating & Negative Review` | Code | Keeps low-rated businesses with bad reviews |
| `Dedupe by place_id` | Code | Removes duplicate businesses |
| `Append or update row in sheet` | Google Sheets | Writes leads to the sheet |

---

## Search Scope

The Apify actor is called with a large `searchStringsArray` of ecommerce queries for **Lahore**, covering clothing, shoes, bags, jewelry, cosmetics, electronics, furniture, groceries, pharmacy, and more.

## Output Fields

| Field | Description |
|-------|-------------|
| `name` | Business name |
| `rating` | Google rating (only `< 3.5` kept) |
| `total_reviews` | Total review count |
| `address` | Business address |
| `website` | Business website |
| `phone` | Business phone |
| `category` | Business category |
| `maps_url` | Google Maps URL |
| `review_snippet` | Text of a negative review |
| `review_count` | Number of reviews fetched |
| `latitude` / `longitude` | Coordinates |
| `working_hours` | Opening hours (JSON) |
| `place_id` | Google Place ID (dedupe key) |

---

## Google Sheet

- **Document:** `Ecommerce negaive reviews businesses`
- **Sheet / tab:** `Sheet1`
- Rows are **appended or updated** using auto-mapped input data.

## Credentials & Services Required

| Credential | Used By |
|------------|---------|
| HTTP Header Auth | Apify actor run |
| HTTP Bearer Auth | Apify actor run |
| Google Sheets OAuth2 | Append or update row in sheet |

External service:

- **Apify** (`compass~crawler-google-places` actor) — requires an Apify API token.

---

## Security & Setup Notes

Sensitive values were redacted from this public copy. Before running, replace them with your own:

- `YOUR_APIFY_API_TOKEN` — your Apify API token (appears in three request URLs)
- `YOUR_GOOGLE_SHEET_ID` — your leads spreadsheet ID
- `REPLACE_WITH_CREDENTIAL_ID` / `REPLACE_WITH_CREDENTIAL_NAME` — attach your own n8n credentials

Recommendations:

- Move the Apify token out of the node URLs and into an **HTTP Header Auth** or **HTTP Bearer Auth** credential.
- The retry loop only checks immediately after a fixed 60 second wait; consider polling with a delay and a max-attempt guard.

Setup steps:

1. Import the JSON into n8n.
2. Attach your Apify (HTTP auth) and Google Sheets credentials.
3. Insert your Apify token into the three Apify request URLs.
4. Point the Google Sheets node at your spreadsheet/tab.
5. Run the workflow manually.
