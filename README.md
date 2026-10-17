# How to Scrape Jobicy: Remote Jobs via the Official Public API

> **Run it now, no code: [jobicy-jobs-scraper on the Apify Store](https://apify.com/bibian/jobicy-jobs-scraper)** &nbsp;|&nbsp; $2.00 per 1,000 job postings &nbsp;|&nbsp; no browser, no login

Jobicy is a large remote-job board with thousands of active postings. It publishes an official public JSON API for third parties, but with no ready-made pagination past the first page or built-in deduplication across runs. This guide shows how to get its postings as clean structured data: title, company, salary, industry, level and the full description, with a free Apify Actor, from the console or from code.

Searches this page answers: jobicy scraper, jobicy API, scrape remote jobs with salary.

## What you get from the Jobicy scraper

- Remote postings filtered by location, industry or skill, exactly as Jobicy's own filters accept.
- Salary kept as numbers (`salaryMin`/`salaryMax`) with currency and period, whenever a posting discloses them.
- Cursor-based pagination handled for you, past Jobicy's own first page, up to `maxItems`.
- A stable `id` and canonical `url` per posting for deduplication across runs.

Export the dataset as JSON, CSV or Excel, or read it through the Apify API. A real sample (five results, personal data removed, long texts truncated) is in [`sample-output.json`](sample-output.json).

## Output fields

| Field | What it is |
|---|---|
| `id`, `url` | Jobicy posting id and public link |
| `title` | Job title |
| `companyName`, `companyLogo` | Hiring company and its logo URL |
| `industry`, `jobType` | Jobicy's own category and employment type, as arrays |
| `geo`, `level` | Location filter value and seniority level |
| `excerpt`, `description` | Short and full plain text (full text only when `includeDescriptions` is set) |
| `salaryMin`, `salaryMax`, `salaryCurrency`, `salaryPeriod` | Salary range as numbers, `null` when not disclosed |
| `publishedAt` | ISO timestamp |

## Use cases

- **Benchmark remote salaries by role or region.** Group `salaryMin`/`salaryMax` by `geo`, `industry` or `level`.
- **Build a daily remote job alert.** Schedule a run filtered by `geo`, `industry` or `tag` and forward new postings to Slack, e-mail or a Google Sheet.
- **Cover large volumes in one run.** Cursor pagination lets a single run walk thousands of postings past the first page.
- **Feed a job board or newsletter.** Use the structured fields and canonical `url` to power a niche remote-work board.

## Input

| Field | What it does |
|---|---|
| `geo` | Keep only jobs matching this location, for example `usa`. Empty means every location. |
| `industry` | Keep only jobs matching this industry, for example `marketing`. Empty means every industry. |
| `tag` | Keep only jobs matching this skill or tag, for example `python`. Empty means no tag filter. |
| `maxItems` | The run stops once this many jobs are saved (default 100). |
| `includeDescriptions` | Adds the full job description as plain text. |

Example input:

```json
{
  "geo": "usa",
  "maxItems": 100,
  "includeDescriptions": true
}
```

## Three ways to use it

### 1. Apify console, no code

1. Open [the Actor page](https://apify.com/bibian/jobicy-jobs-scraper) and click **Try for free** (a free Apify account is enough).
2. Fill in the input form (location, industry, tag and the maximum number of results) and click **Start**.
3. When the run finishes, open the **Output** tab, check the table, then **Export** as CSV, Excel or JSON.
4. For a daily alert, open **Schedules**, create a daily schedule for the Actor and, optionally, add an integration that sends new results to Slack, e-mail or Google Sheets.

### 2. Apify API with JavaScript

```bash
npm install apify-client
export APIFY_TOKEN="your token from Apify console, Settings, Integrations"
node examples/run.js
```

```js
import { ApifyClient } from 'apify-client';

const client = new ApifyClient({ token: process.env.APIFY_TOKEN });

const run = await client.actor('bibian/jobicy-jobs-scraper').call({
    "geo": "usa",
    "maxItems": 100,
    "includeDescriptions": true
});

const { items } = await client.dataset(run.defaultDatasetId).listItems();
for (const item of items) {
    console.log(`${item.title} | ${item.companyName} | ${item.geo} | ${item.salaryMin ?? '-'}-${item.salaryMax ?? '-'} ${item.salaryCurrency ?? ''}`);
}
```

The complete file is [`examples/run.js`](examples/run.js) (ES module, Node 18 or later).

### 3. Apify API with Python

```bash
pip install apify-client
export APIFY_TOKEN="your token from Apify console, Settings, Integrations"
python examples/run.py
```

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])

run = client.actor("bibian/jobicy-jobs-scraper").call(run_input={
    "geo": "usa",
    "maxItems": 100,
    "includeDescriptions": True,
})

for item in client.dataset(run.default_dataset_id).iterate_items():
    print(f"{item['title']} | {item.get('companyName')} | {item.get('geo')} | {item.get('salaryMin')}-{item.get('salaryMax')} {item.get('salaryCurrency') or ''}")
```

The complete file is [`examples/run.py`](examples/run.py). Never put your token in the code: read it from the `APIFY_TOKEN` environment variable as above.

## Pricing

Pay per result: **$2.00 per 1,000 job postings** (that is $0.002 per job posting), plus a tiny start fee of $0.00005 per run. You pay only for the results saved to the dataset. Apify's free plan includes monthly credits, enough to try it on a few hundred results. Prices are those shown on the [Store page](https://apify.com/bibian/jobicy-jobs-scraper) when this guide was written; the Store page is the reference.

## Limits and honest notes

- Jobicy's own `geo`/`industry`/`tag` filters are loose: an unmatched value is ignored rather than rejected, so check a few results against your filter before scaling up a run.
- Not every posting discloses a salary; `salaryMin`/`salaryMax` are `null` when that is the case, nothing is guessed.
- Every e-mail address and phone-like number found in `excerpt` or `description` is replaced by `[removed]`, including generic company contact addresses.
- This is an unofficial tool, not affiliated with Jobicy. Check the API's own terms (crediting Jobicy and linking applicants to the original job URL) and the data protection rules that apply to your use.

## FAQ

**Does it need a browser or a proxy?**  
No. The Actor reads Jobicy's own public JSON API directly.

**Does it work without any filter?**  
Yes, for a productive default run across the newest remote postings.

**Can I get the full description text?**  
Yes, set `includeDescriptions` to `true`.

**Can I get the data in Excel or Google Sheets?**  
Yes. Export the dataset as CSV, Excel or JSON from the Apify console, or connect the Apify Google Sheets integration.

## Try it

**[Open jobicy-jobs-scraper on the Apify Store](https://apify.com/bibian/jobicy-jobs-scraper)**

The code in `examples/` is released under the MIT license (see [LICENSE](LICENSE)). The data belongs to its original publishers.
