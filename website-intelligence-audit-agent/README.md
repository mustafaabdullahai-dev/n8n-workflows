# Website Intelligence Audit Agent

n8n workflow that audits business websites from a Google Sheets lead list, scores them across technical and subjective categories, produces a PDF audit report with an AI-written summary and outreach message, and writes the results back to the sheet.

- **Workflow name:** Website Intelligence Audit Agent
- **File:** `Website Intelligence Audit Agent.json`
- **Status:** Active
- **Execution order:** v1

---

## How It Works

1. A **schedule trigger** starts the run.
2. Rows are read from a Google Sheet and filtered to those whose `Audit Status` is empty (i.e. not yet audited).
3. Rows are processed in a batch loop, one website at a time.
4. For each website:
   - The homepage is fetched. If it is unreachable, the row is marked `Failed: Unreachable`.
   - Google **PageSpeed Insights** is called (mobile; performance, accessibility, SEO, best practices).
   - HTML is parsed for technical signals (title, meta description, OG tags, canonical, headings, schema, robots).
   - All links are HEAD-checked to score broken links.
   - An LLM scores subjective categories (UX, navigation, trust signals, etc.).
   - Scores are merged into an overall score.
   - A second LLM produces the executive analysis and outreach message.
   - An HTML report is built and converted to PDF via APITemplate.io.
   - The PDF is uploaded to Google Drive, shared publicly (reader), and the sheet row is updated with `Completed`, the outreach message, and the PDF link.

## Node Flow

```
Schedule Trigger
  -> Get row(s) in sheet (Google Sheets)
  -> Filter (Audit Status is empty)
  -> Limit
  -> Loop Over Items (Split In Batches)
       -> Fetch Homepage (HTTP)
       -> Site Reachable?
            ├─ true  -> Mark Failed -> Update Row - Failed -> Loop Over Items
            └─ false -> Rate Limit Pause (Wait 2s)
                    -> PageSpeed Insights (HTTP)
                    -> Technical Audit Parser (Code)
                    -> Broken Link Checker (Code)
                    -> Basic LLM Chain (Gemini)
                    -> Merge All Scores (Code)
                    -> Basic LLM Chain1 (Gemini)
                    -> Build HTML Report (Code)
                    -> Generate PDF (APITemplate.io)
                    -> Download PDF (HTTP)
                    -> Upload PDF (Google Drive)
                    -> Share PDF (Google Drive)
                    -> Update Row - Success (Google Sheets)
                    -> Loop Over Items
```

---

## Nodes

| Node | Type | Purpose |
|------|------|---------|
| `Schedule Trigger` | Schedule Trigger | Starts runs |
| `Get row(s) in sheet` | Google Sheets | Reads leads |
| `Filter` | Filter | Keeps rows with empty `Audit Status` |
| `Limit` | Limit | Caps the number of items per run |
| `Loop Over Items` | Split In Batches | Iterates rows one by one |
| `Fetch Homepage` | HTTP Request | Downloads the site homepage HTML |
| `Site Reachable?` | IF | Routes unreachable sites to failure handling |
| `Mark Failed` | Set | Sets `Failed: Unreachable` and reason |
| `Update Row - Failed` | Google Sheets | Writes failure status back |
| `Rate Limit Pause` | Wait | 2 second pause before external calls |
| `PageSpeed Insights` | HTTP Request | Google PageSpeed Insights API (mobile) |
| `Technical Audit Parser` | Code | Regex-parses HTML into technical scores |
| `Broken Link Checker` | Code | HEAD-checks links and scores broken links |
| `Basic LLM Chain` | LangChain LLM Chain | Scores subjective categories |
| `Google Gemini Chat Model` | Gemini Chat Model | Backs `Basic LLM Chain` |
| `Merge All Scores` | Code | Combines technical + AI scores |
| `Basic LLM Chain1` | LangChain LLM Chain | Executive analysis and outreach message |
| `Google Gemini Chat Model1` | Gemini Chat Model | Backs `Basic LLM Chain1` |
| `Build HTML Report` | Code | Renders the HTML report |
| `Generate PDF` | HTTP Request | APITemplate.io PDF generation |
| `Download PDF` | HTTP Request | Downloads the generated PDF |
| `Upload PDF` | Google Drive | Uploads PDF to a Drive folder |
| `Share PDF` | Google Drive | Shares PDF with anyone (reader) |
| `Update Row - Success` | Google Sheets | Writes completed status, message, link |

---

## Google Sheet

- **Document:** `California Non-IT Leads`
- **Sheet / tab:** `Sheet2`

Expected columns:

| Column | Description |
|--------|-------------|
| `Business Type` | Business category |
| `name` | Business name |
| `phone number` | Contact phone |
| `address` | Business address |
| `website` | URL to audit (input) |
| `email` | Contact email |
| `Audit Status` | Empty = pending; set to `Completed` / `Failed: Unreachable` |
| `Outreach Message` | Written by the workflow (output) |
| `PDF Report Link` | Public Drive link to the report (output) |
| `row_number` | Internal row index used for updates |

---

## Credentials & Services Required

| Credential | Used By |
|------------|---------|
| Google Sheets OAuth2 | Get row(s) in sheet, Update Row - Failed / Success |
| Google Drive OAuth2 | Upload PDF, Share PDF |
| Google Gemini (PaLM) API | Google Gemini Chat Model, Google Gemini Chat Model1 |
| HTTP Header Auth | Generate PDF (APITemplate.io API key) |

External APIs called directly by HTTP Request nodes:

- **Google PageSpeed Insights API** (`runPagespeed`, mobile) — requires a Google API key.
- **APITemplate.io** (`/v2/create-pdf`) — requires a template ID and API key via HTTP Header Auth.

## Models Used

- **Text:** Google Gemini (`models/gemini-3.1-flash-lite`)

---

## Security & Setup Notes

Sensitive values were redacted from this public copy. Before running, replace them with your own:

- `YOUR_GOOGLE_SHEET_ID` — your lead spreadsheet ID
- `YOUR_DRIVE_FOLDER_ID` — the Drive folder for reports
- `YOUR_GOOGLE_PAGESPEED_API_KEY` — your PageSpeed Insights API key
- `YOUR_APITEMPLATE_TEMPLATE_ID` — your APITemplate.io template ID
- `REPLACE_WITH_CREDENTIAL_ID` / `REPLACE_WITH_CREDENTIAL_NAME` — attach your own n8n credentials

Setup steps:

1. Import the JSON into n8n.
2. Attach credentials for Google Sheets, Google Drive, Google Gemini, and HTTP Header Auth (APITemplate.io).
3. Insert your Google PageSpeed API key into the `PageSpeed Insights` URL and your template ID into the `Generate PDF` URL.
4. Point the Google Sheets nodes at your spreadsheet/tab and the Drive nodes at your reports folder.
5. Add leads with an empty `Audit Status` and activate the workflow.
