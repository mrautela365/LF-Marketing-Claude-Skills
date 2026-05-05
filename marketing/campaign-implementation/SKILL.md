---
name: campaign-implementation
description: >
  Creates live, paused campaign drafts across Google Ads (Search + Display), Meta Ads
  Manager, and LinkedIn Campaign Manager using browser automation. Requires an approved
  campaign brief (Excel file from campaign-brief-generator). All campaigns are created
  in paused/draft state — no budget is spent until the user explicitly activates them.

  Trigger after the user approves the campaign brief and says: "implement the campaigns",
  "create the campaigns", "set up the ads", or "proceed to implementation".

  Prerequisites: approved campaign brief Excel file, browser logged into each ad platform.
---

# Campaign Implementation

Automates campaign creation across ad platforms using browser automation
(`mcp__Claude_in_Chrome__computer`). Always creates campaigns in paused or draft state.
Never activates a campaign or spends budget without a second explicit user confirmation.

## ⚠️ Safety Rules

1. **Paused by default**: All Google Ads campaigns must be published then immediately paused.
   Never leave a campaign in "Enabled" state after creation.
2. **Draft/paused for Meta and LinkedIn**: Save as draft — do not publish to active.
3. **Budget placeholders**: Use $1.00/day as a placeholder. Real budgets must be filled
   by the campaign stakeholder before activation.
4. **Second approval gate**: Before any campaign goes live (spending real money), stop and
   ask the user for explicit confirmation. This skill only creates drafts.

## Prerequisites

| Requirement | Details |
|-------------|---------|
| Campaign brief | Approved `{event_slug}-campaign-brief.xlsx` |
| Browser auth | Chrome must be logged into Google Ads, Meta Ads Manager, LinkedIn Campaign Manager |
| Tab ID | Use `tabs_context_mcp` to get the active tab ID before starting |

## Configuration

| Setting | Value |
|---------|-------|
| Google Ads account | LF Core (select from account switcher if not already active) |
| Campaign naming pattern | `Events \| {Event Name} {Year} \| {Region} \| Conversions \| Prospecting \| {Platform} \| {Brand} \| BoFU` |
| Default daily budget | `$1.00` (placeholder — stakeholder fills real budget) |
| Default bid strategy | Maximize conversions (no CPA target for new events) |

---

## Platform 1: Google Ads — Search Campaign

### 1a. Navigate to New Campaign

1. Go to `https://ads.google.com/aw/campaigns`
2. Click the blue **+** button → **New campaign**
3. Objective: **Website traffic** → Goal: **Website visits**
4. Campaign type: **Search**

### 1b. Campaign Settings

| Field | Value |
|-------|-------|
| Campaign name | `Events \| {Event Name} {Year} \| {Region} \| Conversions \| Prospecting \| Search \| {Brand} \| BoFU` |
| Networks | Uncheck "Include Google Display Network" and "Include Google Search Partners" (keep only Google Search) |
| Locations | Target: event country (e.g. India). Exclude: none |
| Languages | English |
| Budget | `$1.00` / day (placeholder) |
| Bid strategy | Maximize clicks (or Maximize conversions if conversion tracking is configured) |
| Start/end date | Leave blank — stakeholder sets actual dates |

### 1c. Ad Group + Keywords

Create one ad group named `MCP - High Intent` (or event-appropriate name).

Add all high-intent keywords from the brief's keyword list (🔴 High intent rows).
Use the match type specified in the brief (Exact / Phrase / BMM).

Skip 🟢 Low intent keywords (informational) or add as negatives.

### 1d. Create RSA Ad

Fill in exactly the values from the `Google Search` tab of the brief:
- 15 headlines (add all — Google optimises which to show)
- 10 descriptions
- Final URL: registration URL with UTM parameters from the brief's Overview tab
- Display path: e.g. `events.linuxfoundation.org/register`

### 1e. Add Sitelinks

Add all 6 sitelinks from the brief. Each needs:
- Headline (≤ 25 chars)
- Description line 1 (≤ 35 chars)
- Description line 2 (≤ 35 chars)
- Final URL (with UTM)

### 1f. Publish then immediately pause

1. Click **Publish campaign**
2. Wait for the "Campaign published" confirmation
3. Navigate to the Campaigns list
4. Find the campaign by name
5. Click the green **Enabled** status dot → select **Paused**
6. Confirm the status dot turns grey/paused

> **Why publish-then-pause?** Drafts in the Google Ads wizard are fragile (URL-based,
> lost on navigation). Publishing locks the campaign into the Campaigns list where it
> can be reliably found, edited, and paused. Pausing before any budget period begins
> ensures zero spend.

**Record the campaign ID** from the URL (`campaignId=XXXXXXXXXX`).

---

## Platform 2: Google Ads — Display Campaign

### 2a. Navigate to New Campaign

1. Go to `https://ads.google.com/aw/campaigns`
2. Click **+** → **New campaign**
3. Objective: **Website traffic**
4. Campaign type: **Display**
5. Subtype: **Standard Display campaign**
6. Final URL: registration URL from brief

### 2b. Campaign Settings

| Field | Value |
|-------|-------|
| Campaign name | `Events \| {Event Name} {Year} \| {Region} \| Conversions \| Prospecting \| Display \| {Brand} \| BoFU` |
| Locations | Event country |
| Languages | English |
| Budget | `$1.00` / day |
| Bid strategy | **Maximize conversions** — uncheck "Set a target cost per action" for new events with no conversion history |

### 2c. Targeting

- No keyword targeting needed (Display uses audience signals)
- Optionally add audience segments: In-market for Software, Technology

### 2d. Create Responsive Display Ad

Fill fields from the `Google Display` tab of the brief:

| Field | Value from Brief |
|-------|-----------------|
| Business name | `Linux Foundation Events` (≤ 25 chars) |
| Short headline | From brief (≤ 30 chars) |
| Long headline | From brief (≤ 90 chars) |
| Description | From brief (≤ 90 chars) |
| Final URL | Registration URL + UTM (utm_medium=display) |

**Images — critical:**
- Add images using the event URL scanner (paste registration URL → let Google suggest)
- Accept **landscape (1.91:1)** images — these usually succeed
- ⚠️ **Square (1:1) images often fail auto-crop** from event pages. If "Something went
  wrong. Try cropping again" error appears: dismiss, proceed without square image,
  and flag for Design team
- Campaign cannot be published without at least 1 landscape AND 1 square image
- If square image is missing: **save as draft** and record the campaignId + draftId

### 2e. Save state

If both image formats are available: publish then immediately pause (same as Search).

If square image is missing: save as draft. Record:
- `campaignId` (from URL)
- `draftId` (from URL)
- Note for Design team: needs square (1:1) brand image + logo before publishing

---

## Platform 3: Meta Ads Manager

### 3a. Navigate to Ads Manager

Go to `https://adsmanager.facebook.com/adsmanager/manage/campaigns`

Ensure the correct ad account is selected (check top-left account switcher).

### 3b. Create Campaign

1. Click **+ Create**
2. Objective: **Leads** or **Website conversions**
3. Campaign name: `Events \| {Event Name} {Year} \| {Region} \| Conversions \| Prospecting \| Meta \| {Brand} \| BoFU`
4. Budget: Campaign Budget Optimisation OFF (set at ad set level)
5. Click **Next**

### 3c. Ad Set (Audience + Placements)

**Audience — Prospecting mode (default for new events):**

| Setting | Value |
|---------|-------|
| Locations | Event country + major tech metros |
| Age | 25–55 |
| Interests | Artificial intelligence, Machine learning, APIs, Developer tools, Software development |
| Behaviours | Technology early adopters |
| Custom audiences | None (prospecting) |

> If retargeting lists exist: create custom audience segments from website visitors,
> email list, or past purchasers. ⚠️ Audiences take 2–3 days to populate — create early.

**Placements:** Advantage+ Placements (recommended) or manual: Facebook Feed,
Instagram Feed, Facebook/Instagram Stories.

**Budget:** $1.00/day (placeholder). **Start date:** Leave blank.

### 3d. Ad Creative

| Field | Value |
|-------|-------|
| Ad name | `{Event Name} - Primary` |
| Identity | Select the LF Events Facebook page |
| Format | Single image |
| Image | Upload event image (Design team to provide; or use event banner from registration page) |
| Primary text | Use Variant 1 from the brief's Meta tab |
| Headline | Use Variant 1 from the brief's Meta tab |
| Description | Optional |
| CTA | **Register Now** |
| Website URL | Registration URL + UTM (utm_source=facebook, utm_medium=paid-social) |

**Save as draft** — do not publish.

### 3e. Duplicate for additional variants (optional)

If time permits, duplicate the ad and apply primary text + headline Variants 2 and 3
from the brief for A/B testing.

---

## Platform 4: LinkedIn Campaign Manager

### 4a. Navigate to Campaign Manager

Go to `https://www.linkedin.com/campaignmanager/`

Select the correct account.

### 4b. Create Campaign Group

Click **+ Create** → **Campaign Group**
- Name: `Events | {Event Name} {Year}`
- Status: Paused
- Budget: None at group level

### 4c. Create Campaign

Inside the campaign group, click **+ Create campaign**:

| Setting | Value |
|---------|-------|
| Objective | **Website visits** or **Lead generation** |
| Campaign name | `Events \| {Event Name} {Year} \| {Region} \| Conversions \| Prospecting \| LinkedIn \| {Brand} \| BoFU` |
| LinkedIn Audience Network | Enabled |
| Targeting | See below |
| Format | Single image ad |
| Budget | $1.00/day (placeholder) |
| Schedule | No end date; start date blank |

**Targeting:**

| Dimension | Values |
|-----------|--------|
| Job titles | Software Engineer, AI Engineer, ML Engineer, Solutions Architect, CTO, VP Engineering, Platform Engineer |
| Industries | Information Technology, Financial Services, Computer Software |
| Company size | 50–10,000+ employees |
| Location | Event country |

### 4d. Ad Creative

| Field | Value |
|-------|-------|
| Ad name | `{Event Name} - Primary` |
| Intro text | Variant 1 from brief's LinkedIn tab (≤ 150 chars) |
| Headline | Variant 1 from brief's LinkedIn tab (≤ 70 chars) |
| Image | Upload event image |
| CTA | **Register** |
| Destination URL | Registration URL + UTM (utm_source=linkedin, utm_medium=paid-social) |

**Save as draft.**

---

## Handoff Record

After completing all platforms, output a structured summary:

```
## Campaign Implementation Summary — {Event Name}

### Google Ads Search
- Campaign ID: XXXXXXXXXX
- Status: Paused ✅
- Keywords: {N} loaded

### Google Ads Display
- Campaign ID: XXXXXXXXXX / Draft ID: XXXXXXXXXX
- Status: Draft ⚠️ (needs square image from Design team before publishing)
- Action needed: Upload 1:1 square brand image + logo

### Meta Ads
- Campaign name: Events | {Event Name}...
- Status: Draft ✅
- Account: {account name}

### LinkedIn
- Campaign group: Events | {Event Name} {Year}
- Campaign name: Events | {Event Name}...
- Status: Draft ✅

---
⚠️ BEFORE GOING LIVE — stakeholder must:
  1. Set real budget (replace $1.00/day placeholders)
  2. Set campaign start/end dates
  3. Upload square (1:1) image to Google Display draft
  4. Review all ad copy for brand accuracy
  5. Confirm conversion tracking is firing
  6. Then explicitly say "activate campaigns" for a second approval step
```

**Do not activate any campaign without a second, explicit user instruction.**

---

## Troubleshooting Reference

| Issue | Fix |
|-------|-----|
| Google Ads: "Bidding: Enter an amount" error on Display review | Uncheck "Set a target cost per action" checkbox — use pure Maximize conversions |
| Google Ads: "Ad creation: Fix errors" on Display publish | Missing square (1:1) image — save as draft, flag for Design team |
| Google Ads: Image crop error toast | Dismiss and proceed; auto-crop of event page images fails for square format |
| Google Ads: Accidentally clicked copy/duplicate icon | Click the trash icon on the duplicate → confirm "Yes remove" |
| Meta: Ad account not visible | Check account switcher top-left; may need access granted by Ads manager |
| LinkedIn: Campaign won't save | Ensure targeting has at least one location selected |
