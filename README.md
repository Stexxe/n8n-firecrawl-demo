# Company Research Automation — n8n + Firecrawl + Google Sheets

Reads a list of company URLs from Google Sheets, uses [Firecrawl](https://firecrawl.dev)'s AI extraction to pull structured data from each page, and writes the results to a separate output sheet — fully automated with [n8n](https://n8n.io).

![Workflow overview](assets/workflow.png)

---

## What gets extracted

| Field | Description |
|---|---|
| `url` | Source URL from input sheet |
| `company_name` | Official company name |
| `tagline` | Main value proposition |
| `description` | 2–3 sentence summary of what the company does |
| `industry` | Business sector |
| `headquarters` | City and country |
| `key_products` | Comma-separated list of main products/services |
| `scraped_at` | ISO timestamp of when the row was written |

---

## Sample output

| url | company_name | tagline | industry | headquarters | key_products |
|---|---|---|---|---|---|
| https://stripe.com | Stripe | Financial infrastructure for the internet | Fintech / Payments | San Francisco, USA | Payments API, Billing, Connect, Radar |
| https://notion.so | Notion | The connected workspace | Productivity / SaaS | San Francisco, USA | Docs, Databases, AI, Wikis |
| https://linear.app | Linear | The issue tracker built for modern product teams | Project Management | San Francisco, USA | Issue tracking, Project planning, Roadmaps |

---

## How it works

```
Google Sheets (Sheet1)
  └─ Read URLs
       └─ Loop one at a time
            └─ Firecrawl /v1/extract  ← AI extracts structured fields
                 └─ Format & clean row
                      └─ Append to Results sheet
                           └─ (loop back until all URLs processed)
```

---

## Prerequisites

- **n8n** — self-hosted (free, unlimited) or n8n.cloud
- **Firecrawl** — free account at [firecrawl.dev](https://firecrawl.dev) (500 credits/month, no card needed)
- **Google account** with access to Google Sheets
- **Google Cloud project** with the Google Sheets API enabled (for OAuth2)

---

## Setup

### 1. Prepare Google Sheets

Create a new Google Spreadsheet with two sheets:

**Sheet1** — input (one URL per row, header in row 1):
```
url
https://stripe.com
https://notion.so
https://linear.app
https://vercel.com
https://supabase.com
```
You can import `sample-data/companies.csv` directly or paste the URLs manually.

**Results** — output (create the sheet, leave it empty — the workflow writes the headers automatically on first append if you use `USER_ENTERED` mode, or add them manually):
```
url | company_name | tagline | description | industry | headquarters | key_products | scraped_at
```

Copy your **Spreadsheet ID** from the URL:
```
https://docs.google.com/spreadsheets/d/THIS_IS_YOUR_SPREADSHEET_ID/edit
```

### 2. Start n8n

**Option A — Docker (recommended):**
```bash
docker run -it --rm -p 5678:5678 -v ~/.n8n:/home/node/.n8n n8nio/n8n
```

**Option B — npx:**
```bash
npx n8n
```

Open [http://localhost:5678](http://localhost:5678).

### 3. Import the workflow

1. In n8n, go to **Workflows** → **Add workflow** → top-right menu → **Import from file**
2. Select `workflow.json` from this repo

### 4. Create credentials

**Google Sheets (OAuth2):**
1. In n8n go to **Settings → Credentials → Add credential → Google Sheets OAuth2 API**
2. Follow the [n8n Google Sheets setup guide](https://docs.n8n.io/integrations/builtin/credentials/google/) to create OAuth2 credentials in Google Cloud Console
3. Name the credential **"Google Sheets account"**

**Firecrawl API key:**
1. Get your API key from [firecrawl.dev/app/api-keys](https://www.firecrawl.dev/app/api-keys)
2. In n8n go to **Settings → Credentials → Add credential → HTTP Header Auth**
3. Set **Name** to `Authorization` and **Value** to `Bearer YOUR_API_KEY`
4. Save it as **"Firecrawl API"**

### 5. Configure the workflow

Open the imported workflow and update two nodes:

- **Read Company URLs** node → set your Spreadsheet ID and select your Google Sheets credential
- **Write to Results Sheet** node → set the same Spreadsheet ID and select your Google Sheets credential

Both nodes have a `YOUR_SPREADSHEET_ID` placeholder — replace it with the ID you copied in step 1.

### 6. Run it

Click **Execute Workflow**. Watch the nodes light up one by one. Results appear in your **Results** sheet in real time.

---

## Customization

**Change what gets extracted** — edit the `schema` in the **Firecrawl Extract** node body. Add or remove fields; Firecrawl's AI will find them on the page.

**Change the prompt** — the `prompt` field in the same node guides extraction. Be specific: `"Focus on the pricing page data"` or `"Extract the founding team names"`.

**Add more companies** — just add rows to Sheet1. The loop handles any number of URLs. Keep an eye on your Firecrawl credit balance (1 credit per URL).

**Run on a schedule** — swap the **Start** (Manual Trigger) node for a **Schedule Trigger** node (e.g., every Monday at 9am).

---

## Troubleshooting

**Loop only runs once and stops**
The SplitInBatches node has two outputs: `done` (index 0) and `loop` (index 1). If you reconnect nodes manually, make sure the loop body connects to output index 1 (the bottom handle), not index 0.

**Firecrawl returns empty fields**
Some pages block scrapers. Try adding the company's `/about` page URL instead of the homepage. Also check your Firecrawl credit balance.

**Google Sheets credential error after import**
n8n credential IDs are instance-specific. After importing, open each Google Sheets node and re-select your credential from the dropdown — the placeholder ID won't match your instance.

**`YOUR_SPREADSHEET_ID` error at runtime**
You forgot to replace the placeholder in one of the two Google Sheets nodes. Check both Read and Write nodes.

---

## Cost

| Tool | Free tier | This demo uses |
|---|---|---|
| n8n (self-hosted) | Unlimited | ~10 executions |
| Firecrawl | 500 credits/month | 5 credits (1 per URL) |
| Google Sheets | Free | — |

Total cost: **$0**

---

## License

MIT
