# ARO Dashboard

## Overview

Static dashboard hosted on Netlify, auto-deployed from this GitHub repo. The OpenClaw agent updates results by pushing new data to `data/results.json`.

## How the Agent Updates the Dashboard

After completing a batch run, the agent:

1. Generates the results JSON matching the schema in `data/results.json`
2. Commits and pushes to this repo:
   ```bash
   cd ~/aro-dashboard
   # Write new results
   # (agent writes data/results.json with batch results)
   
   git add data/results.json
   git commit -m "Batch results $(date +%Y-%m-%d_%H%M)"
   git push origin main
   ```
3. Netlify auto-deploys within ~30 seconds
4. Dashboard is live at the Netlify URL

## Data Schema

### `data/results.json`

```json
{
  "batch": {
    "timestamp": "ISO 8601 datetime",
    "total": 65,
    "active": 45,
    "inactive": 17,
    "errors": 3,
    "processing_time": "14m 32s",
    "ai_calls": 8,
    "ai_cost": "$0.024",
    "manual_interventions": 1
  },
  "tasks": [
    {
      "app_id": "string (from HubSpot task)",
      "policy_number": "string or null",
      "status": "active | inactive | error",
      "carrier": "string (carrier name)",
      "plan": "string (product name)",
      "applicant": "string (Last, First)",
      "agents": "string (advisor name(s))",
      "agent_phone": "string (phone for dialer, if available)",
      "surrender_date": "MM/DD/YYYY",
      "age_at_surrender": "string",
      "base_premium": "string (numeric)",
      "account_value": "string ($xx,xxx.xx) or null",
      "surrender_value": "string ($xx,xxx.xx) or null",
      "issue_date": "string (MM/DD/YYYY) or null",
      "documents": ["array of filename strings"],
      "verified": "ISO 8601 datetime",
      "notes": "string (error details or status notes)"
    }
  ],
  "history": [
    {
      "date": "YYYY-MM-DD",
      "total": 65,
      "active": 45,
      "inactive": 17,
      "errors": 3,
      "processing_time": "14m 32s"
    }
  ]
}
```

### Field Requirements

**batch** (required): Summary metrics for the current/most recent batch run.

**tasks** (required): Array of all tasks processed in the current batch. Every task must have:
- `app_id` (always present, from HubSpot)
- `status` (always: "active", "inactive", or "error")
- `carrier` (always present)
- `applicant` (always present)

Fields like `account_value`, `surrender_value`, `issue_date`, and `documents` are only populated for active policies. Set to `null` or `[]` for inactive/error tasks.

**history** (optional but recommended): Array of past batch summaries, most recent first. Append each day's batch summary here to build a running history. Keep the last 30 entries.

### Updating History

When writing results.json, the agent should:
1. Read the existing results.json (if it exists)
2. Preserve the existing `history` array
3. Prepend today's batch summary to `history`
4. Cap history at 30 entries (drop oldest)
5. Replace `batch` and `tasks` with current run data
6. Write the updated file

## Dashboard Features

- **Stats bar**: Total tasks, active, inactive, errors, success rate, total active value
- **Carrier breakdown**: Per-carrier active/inactive/error counts
- **Tabbed task table**: Filter by Active, Inactive, Errors, All
- **Dialer export**: One-click CSV export of active policies for the auto-dialer
- **Batch metrics**: Processing time, AI calls, cost, manual interventions
- **Batch history**: Table of past runs with trend data
- **Auto-refresh**: Dashboard polls for new data every 5 minutes
- **Status indicator**: Shows Live/Stale/Outdated based on data age

## Netlify Setup

1. Create a GitHub repo (e.g., `ka1ro-ai/aro-dashboard`)
2. Push this project to the repo
3. Connect the repo to Netlify (new site from Git)
4. Build settings: no build command needed, publish directory is `/` (root)
5. The site auto-deploys on every push to main

## Local Development

Open `index.html` in a browser. It reads from `data/results.json` relative to the file. No build step, no dependencies, no framework.

## Security Note

This dashboard displays policy numbers, client names, and financial values. Ensure the Netlify site is either:
- Password-protected (Netlify Pro feature), or
- Accessed only via a private/unguessable URL, or
- Restricted via Netlify Identity or similar auth

Do NOT index this site in search engines. Add to `netlify.toml` or `_headers`:
```
X-Robots-Tag: noindex, nofollow
```
