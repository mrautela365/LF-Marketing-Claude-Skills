---
name: campaign-monitor
description: >
  Monitors live campaign performance for any LF event across Google Ads (Search,
  Display, PMax, Demand Gen). Reads metrics directly from the Google Ads UI via
  browser automation — no API key required. Outputs a structured health report in
  chat and appends the snapshot to the campaign state file.

  Trigger when the user says: "check campaign performance", "how are the campaigns
  doing", "pull campaign metrics", "monitor [event]", or "give me a campaign report".

  Prerequisites: campaign state file for the event, browser open and logged into
  Google Ads (ads.google.com). Mumbai campaign must be activated (budget set, status
  enabled) before any metrics are available — paused campaigns show 0 for all metrics.
---

# Campaign Monitor

Reads live Google Ads metrics for any LF event via browser automation and produces
a structured health report. Works with zero external API connections.

## Data Sources & Availability

| Source | What it gives | Status |
|--------|--------------|--------|
| Google Ads UI (browser) | Impressions, Clicks, CTR, Avg CPC, Cost, Conversions, Conv. Rate, Conv. Value | ✅ Always works — primary source |
| HubSpot MCP (`get_campaign_analytics`) | Sessions, New Contacts, Influenced Contacts | ⚠️ Requires HubSpot MCP auth — check first |
| Google Ads MCP (`mcp__google-ads__*`) | Generation only — no reporting tools | ❌ Not available for metrics |

## ⚠️ Pre-flight Checks

Before reading metrics:
1. Confirm campaign status: **paused campaigns show 0 for all metrics** — no data available
2. Check state file for current campaign status. If all campaigns are `paused`, report that and stop.
3. Confirm Google Ads account is LF Core (`mrautela@linuxfoundation.org`, ocid=42259984)
4. If HubSpot MCP is available, note it for Step 5 — otherwise skip to Step 6

## Step 0: Load state file

Read `{event_slug}-campaign-state.json`. Extract:
- `event_name`, `event_slug`, `hs_utm`  
- `campaigns` map — note which platforms have `status: paused` vs `live`/`enabled`

If no state file exists, ask the user for the event name to filter by.

---

## Step 1: Navigate to Google Ads Campaigns view

Navigate to:
```
https://ads.google.com/aw/campaigns?ocid=42259984&__c=3181846416
```

Wait for full page load (Google Ads logo disappears, campaign table appears).

## Step 2: Filter to event campaigns

1. Click the **Search** icon (magnifying glass) in the campaigns table toolbar
2. Type the event name — e.g. `MCP Dev Summit Mumbai`  
3. Press Enter
4. Wait for table to filter

You should see only campaigns matching that event. If 0 rows appear, all campaigns
may be filtered out — try a shorter search term (e.g. just `Mumbai 2026`).

## Step 3: Read campaign-level summary metrics

Take a screenshot and read the header summary row:

| Metric | Where to find it | Notes |
|--------|-----------------|-------|
| Impressions | Top stat bar (blue tile) | 0 = paused or no delivery |
| Cost | Top stat bar | $0 = paused or no spend |
| Conversions | Top stat bar | |
| Conv. value | Top stat bar | |

Then scroll down to the campaigns table. For each campaign row, read:

| Column | Value to capture |
|--------|-----------------|
| Campaign name | Full name |
| Status | Enabled / Paused / Removed |
| Budget | $/day |
| Campaign type | Search / Display / PMax / Demand Gen |

**Scroll right** in the table to reveal performance columns:

| Column | Value to capture |
|--------|-----------------|
| Impr. | Total impressions |
| Clicks | Total clicks |
| CTR | Click-through rate |
| Avg. CPC | Average cost per click |
| Cost | Total spend |
| Conversions | Total conversions |
| Conv. rate | Conversion rate |
| Conv. value | Total conversion value |

> If performance columns are hidden: click **Columns** in the toolbar →
> confirm Impressions, Clicks, CTR, Avg. CPC, Cost, Conversions, Conv. rate are checked.

## Step 4: Drill into each active campaign

For each campaign that is **Enabled** (skip Paused):

1. Click the campaign name
2. Navigate to **Ad groups** view (default on click)
3. Note ad group name and status
4. Click **Ads** in the left sidebar → read individual ad performance
5. Click **Audiences, keywords and content** → **Keywords** → note:
   - Keyword text and match type
   - Quality Score (1–10)
   - Avg. CPC per keyword
   - Top keywords by impressions

6. Click back to campaign level → note any Google recommendations shown

## Step 5: HubSpot attribution (if MCP available)

If `search_crm_objects` / `get_campaign_analytics` tools are connected:

Call `get_campaign_analytics`:
```json
{
  "requests": [{
    "campaignObjectId": {numeric hs_campaign_id from state file},
    "requestedData": "METRICS",
    "startDate": "{30 days ago}",
    "endDate": "{today}"
  }]
}
```

Capture: sessions, new contacts (first touch + last touch), influenced contacts.

If HubSpot MCP is **not** connected: note "HubSpot attribution unavailable — reconnect
HubSpot MCP for session and contact data" in the report.

## Step 6: Read date range

Note the date range currently shown in the Google Ads UI (top right of campaigns view).
Default is "Last 30 days". If the campaign launched recently, switch to "All time":
click the date picker → select "All time" → Apply.

---

## Step 7: Produce health report

Output the following structured report in chat:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 CAMPAIGN HEALTH REPORT — {Event Name}
   Date range: {start} – {end}  |  Pulled: {today}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

CAMPAIGNS

  {Campaign name (short)}
  Type: Search | Status: Paused/Enabled | Budget: $X/day
  Impressions: X,XXX  Clicks: XXX  CTR: X.XX%
  Avg CPC: $X.XX  Cost: $XXX.XX
  Conversions: XX  Conv. Rate: X.XX%  Conv. Value: $XXX

  {Repeat for each campaign}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TOTALS (all campaigns)
  Impressions: X,XXX  Clicks: XXX  Cost: $XXX
  Conversions: XX  CPA: $XX.XX

HUBSPOT ATTRIBUTION (last 30 days)
  Sessions:            XXX
  New contacts (FT):   XX
  New contacts (LT):   XX
  Influenced contacts: XX
  [or: ⚠️ HubSpot MCP not connected]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⚠️  FLAGS & RECOMMENDATIONS

  [List any of the below that apply:]
  • Campaign paused — no data available until activated
  • Budget placeholder ($1.00/day) — stakeholder must set real budget
  • Google Display draft — needs square (1:1) image before publishing
  • CTR below 2% on Search — review headline relevance and keyword match types
  • Quality Score < 7 on brand keywords — review ad relevance and landing page
  • CPA above $X — review bid strategy or budget allocation
  • No conversions after 100+ clicks — check conversion tracking setup
  • [Any Google Recommendations flagged in the UI]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
NEXT ACTIONS
  □ {Specific action with owner}
  □ {e.g. "Activate campaign — stakeholder to set budget"}
  □ {e.g. "Upload square image — Design team"}
  □ {e.g. "Check conversion tracking — Ads manager"}
```

## Step 8: Update state file

Append a `metrics_snapshots` array to `{event_slug}-campaign-state.json`:

```json
{
  "metrics_snapshots": [
    {
      "pulled_at": "2026-05-05",
      "date_range": "2026-04-07 to 2026-05-04",
      "campaigns": {
        "google_search": {
          "status": "paused",
          "budget_day": 1.00,
          "impressions": 0,
          "clicks": 0,
          "cost": 0,
          "conversions": 0
        }
      },
      "hubspot": null
    }
  ]
}
```

---

## Benchmark Reference (LF Events — Search campaigns)

Use these to flag underperforming metrics in Step 7:

| Metric | Flag if | Notes |
|--------|---------|-------|
| CTR (Search) | < 2% | Industry avg for event/brand search is 4–6% |
| CTR (Display) | < 0.3% | Display avg is lower by nature |
| Quality Score | < 7 | Affects ad rank and CPC |
| CPA | > 2× target | Depends on ticket price — set in brief |
| Impression share | < 40% | May need higher bid or budget |
| Conv. rate | < 1% | Check landing page and conversion tracking |

## Known Limitations

| Limitation | Impact | Workaround |
|-----------|--------|-----------|
| Mumbai campaign is paused | All metrics = 0 | Activate campaign first |
| No Google Ads reporting API | Can't pull data programmatically | Browser automation only |
| HubSpot MCP auth expires | Attribution data unavailable | Re-connect HubSpot MCP |
| Google Ads page slow to load | ~5–8s per navigation | Use `wait` before screenshots |
| Metrics columns hidden by default | Missing CTR/CPC data | Click Columns → enable them |
