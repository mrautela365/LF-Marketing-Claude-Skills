---
name: communityseg-to-segment-migration
description: >
  Migrate CommunitySeg (Audienz.ai) audience lists to native HubSpot custom event
  segments. Full end-to-end workflow: identify (CommunitySeg) lists in active HubSpot
  workflows, navigate Audienz.ai to extract Snowflake query logic, then replicate
  that logic as a HubSpot segment using custom events (e.g., "Education Enrolled").

  Trigger on: migrate CommunitySeg list, replicate Audienz audience in HubSpot,
  convert Snowflake query to HubSpot segment, extract query logic from Audienz.ai,
  rebuild audience using custom events, "CommunitySeg migration", "Audienz to HubSpot",
  "custom event segment", "(CS) list", "community seg", "audience migration",
  "move this list to HubSpot natively", "what query does this CommunitySeg list use".
---

# CommunitySeg → HubSpot Custom Event Migration Skill

You are a marketing operations specialist migrating audience segments from the
Audienz.ai / CommunitySeg platform to native HubSpot segments using custom events.
This is a critical workflow because the CommunitySeg lists are currently powered by
Snowflake queries that sync data into HubSpot, but the goal is to replicate that
same logic using HubSpot's built-in custom events — making the segments self-contained
within HubSpot and removing the dependency on the external Audienz.ai platform.

**Before starting, read the worked example at `references/worked-example-dent.md`** —
it walks through a complete migration (DENT - Training Courses) with every click,
URL, and decision documented. Use it as your reference throughout.

## Why This Migration Matters

CommunitySeg (also called Audienz.ai or LFX CommunitySeg) is an audience management
platform that runs Snowflake queries against the Linux Foundation data lake and syncs
the results into HubSpot as contact lists. These lists appear in HubSpot with a
"(CommunitySeg)" prefix in their name. While this system works, it creates a
dependency on an external platform. By migrating to native HubSpot custom events, the
marketing team gains direct control, better visibility into segment logic, and
eliminates a potential point of failure.

---

## Prerequisites

Before starting, ensure you have access to:

1. **HubSpot** (your portal ID) — for viewing workflows, lists, and creating segments
2. **Audienz.ai** (https://app.audienz.ai) — for extracting the Snowflake query logic
3. **Chrome browser tools** — this workflow involves navigating both HubSpot and Audienz.ai
4. **The spreadsheet** (if provided) — the "Workflow Migration Summary" spreadsheet maps
   list IDs to workflow names and indicates which are active (Enabled = "Yes")

If the user hasn't provided a specific list to migrate, ask which CommunitySeg list
or workflow they want to work with.

---

## The Migration Workflow — 6 Steps

### Step 1: Identify the CommunitySeg List in HubSpot

The starting point is always a HubSpot list with "(CommunitySeg)" in its name. You
need to identify:

- **List Name**: The full name, e.g., "(CommunitySeg) DENT - Training Courses"
- **List ID**: The numeric ID from HubSpot (visible in the URL)
- **Associated Workflow**: Which workflow uses this list, and whether it's active

**How to find this information:**

- If the user provides a spreadsheet (like "Workflow Migration Summary"), filter
  Column E (Enabled) for "Yes" to find active workflows. The columns are:
  - A = List ID
  - B = List Name
  - C = Workflow ID
  - D = Workflow Name
  - E = Enabled (Yes/No)
  - F = Flow Type
  - G = Usage Location
  - H = Action ID
  - I = Action Label
  - J = Operator
  - K = Updated At

- If no spreadsheet is provided, navigate to HubSpot:
  - Go to `https://app.hubspot.com/contacts/{portalId}/objectLists`
  - Search for "(CommunitySeg)" to find all synced lists
  - Or go to Automation → Workflows and search for the workflow name

- To verify a workflow is active, navigate to:
  `https://app.hubspot.com/workflows/{portalId}/flow/{workflowId}/edit`
  An active workflow will show "ON" status in the top-right corner.

**What to record:**
```
List Name: (CommunitySeg) [Name]
List ID: [number]
Workflow Name: [name]
Workflow ID: [number]
Workflow Status: Active / Inactive
Usage: enrollmentCriteria / action / etc.
```

### Step 2: Open the CommunitySeg List in HubSpot

Navigate to the list detail page to confirm its existence and see the current
member count:

```
https://app.hubspot.com/contacts/{portalId}/objectLists/{listId}
```

Take note of:
- **Current member count** — you'll compare this against the new segment later
- **List type** — Active or Static
- **Last updated date**

This gives you a baseline to validate against when you create the replacement segment.

### Step 3: Find the List in Audienz.ai

This is the trickiest step because the Audienz.ai UI is a single-page application
that can be slow to load and has some navigation quirks.

**Navigation path:**

1. Go to `https://app.audienz.ai`
2. Wait for the page to fully load (the SPA can take 5-10 seconds; you'll see a
   blue screen with a progress bar while loading — wait for this to complete)
3. In the **left sidebar**, look for the **"Connect"** section
4. Under "Connect", click on **"Audience"** — this is the sub-menu item, NOT the
   top-level tab. The left sidebar structure looks like:
   ```
   Connect
     ├── Audience    ← Click this one
     ├── Folder
     └── ...
   ```
5. Once on the Audience page, use the **search bar** at the top to search for the
   list name (without the "(CommunitySeg)" prefix)
   - For example, if the HubSpot list is "(CommunitySeg) DENT - Training Courses",
     search for "DENT"
   - **Important**: The search has a delay of ~5-8 seconds. After typing your search
     term and pressing Enter, wait patiently for results to appear.

**Common pitfalls:**
- The page may appear blank with just a blue background — this means the SPA
  is still loading. Wait 5-10 seconds.
- Searching in the "Folder" tab instead of the "Audience" tab will return 0 results.
- The search doesn't filter instantly — give it several seconds after typing.
- If results don't appear, try clearing and re-entering the search term.

### Step 4: Extract the Snowflake Query Logic

Once you've found the audience in Audienz.ai, you need to extract the exact query
logic that powers it.

**How to access the query:**

1. In the audience list, find the row for your target audience
2. Look for the **Action column** on the right side of the table — you may need to
   scroll horizontally to see it, as the table can extend beyond the viewport
3. Click the **action icon** (a circular avatar/menu icon) in the Action column
4. From the menu that appears, select **"View Query"** or **"Edit Query"**
5. This opens the query editor showing the Snowflake SQL

**Understanding the query structure:**

Audienz.ai queries typically follow this pattern:
```sql
SELECT EN.USER_EMAIL
FROM ANALYTICS.SILVER_FACT.[TABLE_NAME] AS EN
WHERE (EN.[COLUMN] IN(
  SELECT EN.[COLUMN]
  FROM ANALYTICS.SILVER_FACT.[TABLE_NAME] AS EN
  WHERE EN.[FIELD] ILIKE '%[value1]%'
  OR EN.[FIELD] ILIKE '%[value2]%'
  OR EN.[FIELD] ILIKE '%[value3]%'
))
```

**Key things to extract:**
- **Source table**: Which Snowflake table is being queried (e.g., `ENROLLMENTS`,
  `EVENT_REGISTRATIONS`, `CERTIFICATIONS`)
- **Filter field**: Which column is being filtered on (e.g., `COURSE_NAME`,
  `EVENT_NAME`, `CERTIFICATION_NAME`)
- **Filter values**: The exact values being matched (e.g., specific course names)
- **Match type**: Whether it uses `ILIKE '%...%'` (contains), `=` (exact match),
  or other operators

**Record the full query** — you'll need it to map to HubSpot custom events.

### Step 5: Map Snowflake Logic to HubSpot Custom Events

This is the translation step. You need to convert the Snowflake query into HubSpot
segment filter logic using custom events.

**Mapping table — Snowflake tables to HubSpot custom events:**

| Snowflake Table | HubSpot Custom Event | Key Property |
|---|---|---|
| ANALYTICS.SILVER_FACT.ENROLLMENTS | Education Enrolled | course_name |
| ANALYTICS.SILVER_FACT.EVENT_REGISTRATIONS | Event Registered | event_name |
| ANALYTICS.SILVER_FACT.CERTIFICATIONS | Certification Attempted | certification_name |

(This mapping may evolve over time — if you encounter a Snowflake table not listed
above, check the HubSpot custom events list under Events → Custom Events to find
the matching event.)

**Mapping Snowflake operators to HubSpot filter operators:**

| Snowflake | HubSpot Equivalent |
|---|---|
| `ILIKE '%value%'` | "contains any of" |
| `= 'value'` | "is equal to any of" |
| `NOT ILIKE '%value%'` | "doesn't contain any of" |
| `IN ('val1', 'val2')` | "is equal to any of" (with multiple values) |
| `OR` between conditions on same field | Add all values to the same "contains any of" filter |
| `AND` between conditions on different fields | Add separate filter rows joined by "and" |

**Example translation:**

Snowflake query:
```sql
WHERE EN.COURSE_NAME ILIKE '%Linux Networking and Administration (LFS211)%'
OR EN.COURSE_NAME ILIKE '%Business Considerations for Edge Computing (LFS113x)%'
OR EN.COURSE_NAME ILIKE '%Introduction to Open Source Networking Technologies (LFS165x)%'
```

HubSpot segment:
```
Event: Education Enrolled → has been completed
Frequency: is any number of times
Timeframe: is anytime
course_name → contains any of →
  - Linux Networking and Administration (LFS211)
  - Business Considerations for Edge Computing (LFS113x)
  - Introduction to Open Source Networking Technologies (LFS165x)
```

### Step 6: Create the HubSpot Segment

Now create the actual segment in HubSpot using the mapped logic.

**Naming convention:**
Use the prefix `(CS)` instead of `(CommunitySeg)` to indicate this is a
CommunitySeg-migrated list built with custom events. For example:
- Original: `(CommunitySeg) DENT - Training Courses`
- New: `(CS) DENT - Training Courses`

**Step-by-step in HubSpot:**

1. Navigate to: `https://app.hubspot.com/contacts/{portalId}/objectLists/create?objectTypeId=0-1`

2. **Set the segment name**: Click the pencil icon next to "Untitled segment" at the
   top and enter the new name with the `(CS)` prefix.

3. **Add the event filter**:
   a. Click **"+ Add filter"**
   b. Select the **"Events"** tab (not Properties or Memberships)
   c. **Do NOT search directly** in the top-level Events search — it won't find
      custom events. Instead:
      - Browse the event categories listed below the search box
      - Under "Marketing interactions", find and click **"Custom Events"**
      - This expands to show all available custom events
      - Now use the **"Search Custom Events"** search box to find your event
        (e.g., type "Education Enrolled")
      - Click on the matching event name

4. **Configure the event filter**:
   - The event defaults to "has been completed" — keep this
   - Frequency: "is any number of times" — keep this default
   - Timeframe: "is anytime" — keep this default

5. **Add the property filter**:
   a. Click **"+ Add filter"** within the event block (the inner one, below
      the Frequency/Timeframe row — not the outer "and + Add filter")
   b. This opens a dropdown of **Event properties**
   c. Find and click the relevant property (e.g., **course_name**)
   d. Change the operator:
      - Click the operator dropdown (defaults to "is equal to any of")
      - Select the appropriate operator based on your mapping (e.g., "contains any of")
   e. Add the values:
      - Click the **"Add values"** dropdown
      - In the text input that appears, type each value and press **Enter** to add it
      - Repeat for each value from the Snowflake query
      - Each value appears as a tag/chip in the filter

6. **Verify the preview**:
   - Look at the **Estimated Size** on the right panel
   - It should show a reasonable number of contacts
   - If it shows 0 or "empty", double-check:
     - Did you select the right custom event?
     - Are the property values spelled correctly?
     - Is the operator correct (contains vs. equals)?
   - Compare the estimated count against the original CommunitySeg list member count
     from Step 2. They won't match exactly (the custom event may have more or fewer
     contacts depending on data freshness), but they should be in the same ballpark.

7. **Save the segment**:
   a. Click **"Next"** in the top-right corner
   b. On the "Review and save" panel:
      - Verify "Estimated segment size" looks reasonable
      - Choose processing type: **Active** (so it auto-updates as new contacts match)
      - Optionally add a description explaining this is a migrated CommunitySeg list
      - Segment location: "All segments" is fine, or change folder if your team
        has a specific folder structure
   c. Click **"Save and process segment"**
   d. Wait for processing to complete — the segment page will show
      "Your segment is partially complete" while it processes

8. **Record the new segment details**:
   ```
   New Segment Name: (CS) [Name]
   New Segment ID: [from URL after saving, e.g., objectLists/{newListId}]
   Processing Type: Active
   Estimated Size: [number] contacts
   Original CommunitySeg List Size: [number] contacts
   Delta: [difference]
   ```

---

## Validation Checklist

After creating the new segment, verify:

- [ ] Segment name follows the `(CS)` prefix convention
- [ ] Event type matches the Snowflake source table (e.g., Education Enrolled ↔ ENROLLMENTS)
- [ ] Property name matches the Snowflake filter column (e.g., course_name ↔ COURSE_NAME)
- [ ] Operator matches the Snowflake pattern (ILIKE '%...%' → contains any of)
- [ ] All filter values from the Snowflake query are included
- [ ] Segment is set to "Active" processing type
- [ ] Estimated size is in a reasonable range compared to the original list
- [ ] The original workflow's enrollment criteria can be updated to use the new segment
  (this is a separate step — do NOT update the workflow without explicit approval)

---

## Troubleshooting

### Audienz.ai won't load
- The SPA can take 10+ seconds to render. Wait for the progress bar to complete.
- If stuck on a blue screen, try refreshing the page.
- Make sure you're authenticated — the user must be logged into Audienz.ai in their browser.

### Can't find the audience in Audienz.ai
- Make sure you're searching in the **Audience** tab under **Connect**, not in Folder or elsewhere.
- Try shorter search terms (e.g., "DENT" instead of "DENT - Training Courses").
- The search has a 5-8 second delay — wait after pressing Enter.

### Custom event not found in HubSpot
- Don't use the top-level Events search — it won't find custom events.
- Navigate: Events tab → scroll to "Custom Events" → click to expand → then search.
- If the event still doesn't appear, it may not exist yet — check with the data team.

### Segment shows 0 contacts after saving
- Verify the custom event property values match what's actually in the data.
- Check if the operator is correct — "contains" is more forgiving than "is equal to".
- The segment may still be processing — wait a few minutes and refresh.
- Compare against the original list — if the original also has very few contacts,
  this might be expected.

### Contact counts don't match between old and new
- Some variance is expected. The CommunitySeg list syncs from Snowflake on a schedule,
  while the HubSpot custom event fires in real-time (or near-real-time).
- If the new segment has significantly MORE contacts, the "contains" operator may be
  matching too broadly — consider switching to "is equal to any of" with exact values.
- If significantly FEWER, the property values may not match exactly — check for
  differences in casing, spacing, or special characters.

---

## Batch Migration Notes

When migrating multiple lists:

1. Work through them one at a time — each list may use a different custom event type.
2. Keep a migration tracker with columns:
   - Original List ID | Original List Name | Workflow Name | New Segment ID | New Segment Name | Status
3. After all segments are created, coordinate with the workflow owner before swapping
   the enrollment criteria from the old CommunitySeg list to the new (CS) segment.
4. Consider running both lists in parallel for a week to validate parity before
   decommissioning the old CommunitySeg list.

---

## Quick Reference — Common Custom Events and Properties

| Custom Event | Common Properties |
|---|---|
| Education Enrolled | course_name, course_code, course_slug, course_id, course_group_id |
| All Past Event Registrations | event_name, event_id |
| Certification Attempted | certification_name |
| All Bevy Registrations | event_name |
| Audience Entered | audience_name |
| All Historical Education Enrollments | course_name |

To see the full list of custom events available in your HubSpot portal, go to:
Events tab → Custom Events → browse the full list. Each event has its own set of
properties that can be used as filters.
