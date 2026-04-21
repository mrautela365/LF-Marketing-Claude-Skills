# Worked Example: DENT - Training Courses Migration

This is a complete worked example of migrating the "(CommunitySeg) DENT - Training
Courses" list from Audienz.ai to a native HubSpot custom event segment. Use this
as a reference when performing your own migrations.

---

## Step 1: Identify the List

**Source**: Workflow Migration Summary spreadsheet, filtered to Enabled = "Yes"

| Field | Value |
|---|---|
| List ID | {sourceListId} |
| List Name | (CommunitySeg) DENT - Training Courses |
| Workflow ID | {workflowId} |
| Workflow Name | 24Q2 - DENT - Membership Nurture |
| Enabled | Yes |
| Flow Type | WORKFLOW |
| Usage Location | enrollmentCriteria.listFilterBranch |
| Operator | IN_LIST |

The workflow uses this list as an enrollment criterion — contacts on this list
get enrolled into the DENT membership nurture workflow.

---

## Step 2: Confirm in HubSpot

Navigated to: `https://app.hubspot.com/contacts/{portalId}/objectLists/{sourceListId}`

Confirmed the list exists and noted the current member count for later comparison.

---

## Step 3: Find in Audienz.ai

1. Went to `https://app.audienz.ai`
2. Waited ~8 seconds for the SPA to fully render
3. Clicked "Audience" under the "Connect" section in the left sidebar
4. Searched for "DENT" in the audience search bar
5. Waited ~8 seconds for search results to appear
6. Found: **"Dent - Training Courses"** in the audience list

**Key navigation note**: The search is in the Audience sub-tab, NOT the Folder tab
or the main Connect tab. The left sidebar hierarchy is: Connect → Audience.

---

## Step 4: Extract the Query

1. Scrolled right in the audience table to find the Action column
2. Clicked the action icon (circular avatar) for "Dent - Training Courses"
3. Selected "Edit Query" from the dropdown menu
4. Extracted the following Snowflake query:

```sql
SELECT EN.USER_EMAIL
FROM ANALYTICS.SILVER_FACT.ENROLLMENTS AS EN
WHERE (EN.ENROLLMENT_ID IN(
  SELECT EN.ENROLLMENT_ID
  FROM ANALYTICS.SILVER_FACT.ENROLLMENTS AS EN
  WHERE EN.COURSE_NAME ILIKE '%Linux Networking and Administration (LFS211)%'
  OR EN.COURSE_NAME ILIKE '%Business Considerations for Edge Computing (LFS113x)%'
  OR EN.COURSE_NAME ILIKE '%Introduction to Open Source Networking Technologies (LFS165x)%'
))
```

**Analysis:**
- Source table: `ANALYTICS.SILVER_FACT.ENROLLMENTS`
- Filter field: `COURSE_NAME`
- Match type: `ILIKE '%...%'` (case-insensitive contains)
- Three course names connected by OR logic
- The three DENT-related courses are:
  1. Linux Networking and Administration (LFS211)
  2. Business Considerations for Edge Computing (LFS113x)
  3. Introduction to Open Source Networking Technologies (LFS165x)

---

## Step 5: Map to HubSpot

| Snowflake | HubSpot |
|---|---|
| Table: ENROLLMENTS | Event: Education Enrolled |
| Field: COURSE_NAME | Property: course_name |
| Operator: ILIKE '%...%' | Operator: contains any of |
| OR logic | Multiple values in same filter |

---

## Step 6: Create the Segment

1. Navigated to: `https://app.hubspot.com/contacts/{portalId}/objectLists/create?objectTypeId=0-1`

2. Named the segment: **(CS) DENT - Training Courses**

3. Added filter:
   - Clicked "+ Add filter"
   - Selected "Events" tab
   - Browsed to "Custom Events" (under Marketing interactions)
   - Searched for "Education Enrolled" within Custom Events
   - Clicked "Education Enrolled"

4. Configured event filter:
   - Event: Education Enrolled → has been completed
   - Frequency: is any number of times
   - Timeframe: is anytime

5. Added property filter:
   - Clicked "+ Add filter" (inner, within event block)
   - Selected "course_name" from Event properties
   - Changed operator from "is equal to any of" → **"contains any of"**
   - Added three values (typed each and pressed Enter):
     - `Linux Networking and Administration (LFS211)`
     - `Business Considerations for Edge Computing (LFS113x)`
     - `Introduction to Open Source Networking Technologies (LFS165x)`

6. Verified preview:
   - Estimated size: **6,517 contacts (0.13% of database)**
   - Preview showed real contact names — confirmed logic is working

7. Saved:
   - Clicked "Next"
   - Processing type: **Active**
   - Clicked "Save and process segment"

**Result:**
```
New Segment Name: (CS) DENT - Training Courses
New Segment ID: {newListId}
URL: https://app.hubspot.com/contacts/{portalId}/objectLists/{newListId}
Processing Type: Active
Estimated Size: 6,517 contacts
Created: 2026-04-09
```

---

## Lessons Learned

1. **Audienz.ai search is slow** — always wait 5-8 seconds after searching.
2. **Custom events are nested** — you can't find them by searching in the top-level
   Events search. You must click into "Custom Events" first, then search.
3. **Use "contains any of"** for ILIKE queries — this is the closest HubSpot equivalent
   to Snowflake's `ILIKE '%value%'` pattern.
4. **Name with (CS) prefix** — distinguishes migrated segments from both the original
   (CommunitySeg) lists and other HubSpot segments.
5. **The inner "+ Add filter"** — when adding property filters within an event, use the
   inner add filter button (below the Frequency/Timeframe row), not the outer one
   that adds a separate filter group.
