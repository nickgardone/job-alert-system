# PM Job Monitor

Monitors target companies for new Product Manager job postings and sends email notifications when new roles appear.

## How it works

Runs daily at 9 AM UTC via GitHub Actions. Checks each company's job board API, compares against previously seen roles in `seen_jobs.json`, and emails you when new PM roles are posted.

**Target companies:**
- Airbnb (Greenhouse API)
- Apollo.io (Greenhouse API)
- Salesforce (Workday API)
- Superhuman (Ashby API)
- Playlist (Greenhouse API — three boards: playlist, mindbody, classpass)

## Setup

1. Fork or clone this repo
2. Add a repository secret: **`GMAIL_APP_PASSWORD`** — a [Gmail App Password](https://support.google.com/accounts/answer/185833) for the sending account
3. Update `NOTIFY_EMAIL` in `.github/workflows/check_jobs.yml` to your address
4. Run the seed step to baseline current roles (no alerts for existing jobs):
   ```bash
   pip install -r requirements.txt
   python check_jobs.py --seed
   git add seen_jobs.json && git commit -m "chore: seed seen_jobs" && git push
   ```

## Adding a new company

Always intercept the company's actual careers page network traffic before writing any code — do not guess based on ATS name. Use Playwright to load the page, capture API calls, and identify the exact endpoint and credentials:

```python
from playwright.sync_api import sync_playwright
ATS_KEYWORDS = ["greenhouse", "lever", "ashby", "workday", "myworkdayjobs"]
# capture all responses whose URL contains an ATS keyword, log URL + JSON body
```

Then add an entry to `companies.json` with the appropriate `type` and credentials, and run `--seed` before pushing.

**Supported fetcher types in `companies.json`:**
- `greenhouse` → `board_token` (single) or `board_tokens` (array, for companies with multiple boards)
- `workday` → `api_url`, `base_url`, `search_text` fields
- `ashby` → `org_name` field

## Files

| File | Purpose |
|---|---|
| `check_jobs.py` | Main script — fetches jobs, diffs against seen, sends email |
| `companies.json` | Company configuration (ATS type + credentials) |
| `seen_jobs.json` | State file — job IDs already notified about (auto-committed) |
| `requirements.txt` | Python dependencies |
