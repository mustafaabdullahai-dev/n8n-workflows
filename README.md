# n8n Workflows

A collection of n8n automation workflows. Each workflow lives in its own folder with the exported JSON and a README describing what it does, how it is triggered, and the credentials/services it needs.

## Workflows

| Workflow | Description |
|----------|-------------|
| [LinkedIn Post Automation with AI Image](./linkedin-post-automation-ai-image) | Generates and publishes 3 LinkedIn posts per week with AI-generated images, driven by a Google Sheets content calendar. |
| [Website Intelligence Audit Agent](./website-intelligence-audit-agent) | Audits business websites, scores them across technical and subjective categories, and produces a PDF report with an AI-written summary. |
| [California Non-IT Leads](./california-non-it-leads) | Discovers local non-IT businesses across California via Serper.dev Google Places, extracts contact emails, and appends leads to a Google Sheet. |
| [Finding Ecommerce Business through Agent (Scraping)](./finding-ecommerce-business-agent-scraping) | Uses Apify's Google Places crawler to find low-rated ecommerce businesses in Lahore and saves them to a Google Sheet. |

## Usage

1. Open the folder for the workflow you want.
2. Import the `.json` file into n8n.
3. Reconnect the credentials and replace any redacted values (API keys, sheet IDs, folder IDs) noted in that workflow's README.

## Notes

Credentials and secrets have been redacted from the JSON files in this repository. Follow each workflow's README to supply your own credentials and configuration before running.
