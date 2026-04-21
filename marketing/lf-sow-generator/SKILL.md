---
name: lf-sow-generator
description: |
  Generate Statement of Work (SOW) documents for Linux Foundation Marketing Operations contractor roles by editing a Google Doc template. Covers common LF Marketing Ops roles: Marketing Data Engineer, Performance Marketing, Marketing Operations, and similar contractor engagements with HighGrowth or other vendors.

  Use this skill whenever the user asks to: "create a SOW", "generate a statement of work", "make an SOW for [contractor name]", "draft a new SOW", "new contractor SOW", "SOW for HighGrowth", "create an exhibit for [name]", or mentions needing a contractor agreement for any LF Marketing Ops role. Also trigger when someone says "copy the SOW template" or "new exhibit" or references contractor onboarding docs. Even if the user just says something like "I need to bring on a new contractor" or "set up [name] with a contract", this skill applies.
---

# LF Marketing Ops SOW Generator

This skill generates Statement of Work documents for Linux Foundation Marketing Operations contractor engagements. It works by having the user copy an existing SOW Google Doc template, then systematically editing the copy via Chrome Find & Replace to customize it for the new contractor.

## Prerequisites

- Chrome browser access (Claude in Chrome MCP tools)
- An existing LF SOW Google Doc to use as template (the user should copy one first)
- The user needs to provide: contractor name, role, vendor company, rate, duration, and scope

## Workflow

### Step 1: Gather Information

Ask the user for the following details. If they've already provided some in their message, don't re-ask — just confirm what you have and ask for what's missing.

**Required:**
- Contractor full name (e.g., "Jane Doe")
- Vendor/company name (e.g., "HighGrowth")
- Role type — one of the predefined roles below, or a custom scope
- Number of resources and employment basis (full-time, part-time/daily hours)
- Monthly rate and contract duration (to calculate NTE)
- Start date and end date
- Exhibit number (e.g., A-3, A-4)

**Optional (have sensible defaults):**
- Reporting manager (defaults to the team's Marketing Ops lead)
- Whether to include the full quality metrics framework (default: yes)

### Step 2: Template Setup

Ask the user to:
1. Make a copy of an existing SOW Google Doc (provide the link if they have one, or use the standard template)
2. Share the link to the new copy

If the user has already done this (e.g., "I copied the original here: [link]"), proceed directly.

### Step 3: Edit the Google Doc

Open the user's copied Google Doc in Chrome and use Find & Replace (Cmd+Shift+H / Ctrl+Shift+H) to make all necessary changes. The key principle: **always use the exact text visible in the document** for Find fields. Don't guess at wording — screenshot or read the page to confirm exact text before replacing.

#### Editing sequence:

**a) Exhibit number**
- Find the old exhibit (e.g., "Exhibit A-2") → Replace with new one (e.g., "Exhibit A-3")

**b) Document title**
- Update the Google Doc title to: "STATEMENT OF WORK : [Vendor] : FY[Year] : [Contractor Name]"

**c) Timeframe**
- Replace start date and end date with the new values

**d) Compensation**
- Replace NTE amount
- Replace the resource listing (remove old names, add new contractor with employment basis)
- If going from multiple resources to one, remove extra bullet points by selecting and deleting

**e) Section A — Role-Specific Scope**
Section A is the role-specific deliverable section. Replace the heading and all numbered items/sub-bullets based on the role type. See the Role Templates section below for the exact content per role.

**f) Roles & Responsibilities**
- Update resource count references (e.g., "two resources" → "one full-time resource (Name)")
- Update any name-specific references

**g) Timeline section**
- Update milestones to match the new role's scope and dates

**h) Section numbering**
- Verify all section numbers are sequential with no duplicates (the original template may have duplicate "5" sections for Confidentiality and Timeline)

### Step 4: Verify

After all edits, fetch the Google Doc content via the Google Drive MCP tool and review for:
- No leftover references to the old contractor names
- Correct NTE amount and date range
- Section numbering is sequential
- Role-specific content in Section A matches the selected role template
- Resource count is correct throughout

## Role Templates

These define what goes in **Section A** for each role type. Sections B (HubSpot Ops), C (KPI Dashboards), and D (Documentation) stay the same across all roles.

### Performance Marketing

**Heading:** A. Performance Marketing Automation & Optimization

**HighGrowth will:**

1. Own end-to-end performance marketing operations across all paid digital channels, including:
   - Google Ads (Search, Display, PMax, YouTube)
   - LinkedIn Ads (Sponsored Content, Message Ads, Text Ads)
   - Meta Ads (Facebook and Instagram), Reddit Ads, X/Twitter Ads, and Brave Ads
   - Feathr and Microsoft Ads
2. Build, optimize, and scale paid campaigns across all channels, including audience targeting, bid management, creative rotation, A/B testing, and budget pacing.
3. Design and implement automated campaign workflows, including lead scoring triggers, retargeting sequences, conversion tracking, and cross-channel attribution.
4. Perform QA on all campaign launches, including UTM governance, pixel/tag verification, landing page validation, audience list accuracy, and ad copy compliance.
5. Monitor, report on, and continuously optimize campaign ROAS, CPA, CTR, and conversion rates across all active channels.

**Timeline milestone:** Performance Marketing ramp-up and initial campaign launches: Q2 [Year]

### Marketing Data Engineer

**Heading:** A. Marketing Data Engineering & Pipeline Development

**HighGrowth will:**

1. Design, build, and maintain data pipelines connecting marketing platforms (HubSpot, Segment, Snowflake, Ad Platforms) to enable unified reporting and attribution, including:
   - ETL/ELT pipeline development and orchestration
   - Data warehouse schema design and optimization (Snowflake)
   - Segment source/destination configuration and event modeling
   - Cross-platform identity resolution and data stitching
2. Build and maintain automated data transformations for marketing KPIs, including campaign attribution, funnel metrics, audience scoring, and engagement tracking.
3. Develop and enforce data governance standards across marketing systems, including naming conventions, schema documentation, data quality checks, and access controls.
4. Create monitoring and alerting for data pipeline health, freshness, and accuracy, with automated incident detection and escalation.
5. Support ad-hoc data requests from Marketing Ops, including custom queries, data exports, and troubleshooting data discrepancies across platforms.

**Timeline milestone:** Data pipeline architecture and initial integrations: Q2 [Year]

### Marketing Operations (General)

**Heading:** A. Migration from CommunitySeg to Twilio Segment

(This is the default/original Section A from the template. Keep as-is if the contractor is doing the same migration work as the original SOW.)

### Custom Role

If the user describes a role that doesn't match the above, ask them to describe the key deliverables and draft Section A content collaboratively. Structure it as 5 numbered items with the first item having 4 sub-bullets listing specific technologies or domains.

## Quality Framework

The full quality framework (Sections 3.1–3.7) should be included by default. These sections are role-agnostic and apply to all contractor engagements:

- 3.1 Completeness & First-Time Quality
- 3.2 Data Integrity
- 3.3 Automation Reliability
- 3.4 Documentation Quality
- 3.5 Reusability & Scalability
- 3.6 Timeliness & Communication
- 3.7 Stakeholder Satisfaction

**Note on 3.1 Functional Accuracy:** The original template includes a "3.1 Functional Accuracy" section with Segment mapping-specific metrics. This should only be included if the contractor role involves Segment migration work. For other roles, remove this section and renumber 3.2–3.8 down to 3.1–3.7.

## NTE Calculation

The Not-To-Exceed amount is calculated as: `monthly_rate × number_of_months`

Common patterns:
- Full-time resource at $6K/month for 9 months = $54,000
- Full-time resource at $7.5K/month for 12 months = $90,000
- Two resources (one FT, one PT) for 12 months = varies based on rates

Always confirm the NTE with the user before editing. The Compensation section should list each resource with their basis (Full-Time or part-time hours).

## Find & Replace Tips

Google Docs Find & Replace can be finicky. Key tips for reliable replacements:

1. **Always screenshot the document first** to see the exact text before attempting a replacement
2. **Use the exact text** from the document — don't assume wording from memory
3. **After setting both fields, click "Next" first** to confirm a match is found (check the "X of Y" counter), then click "Replace all"
4. **Work from highest numbers down** when renumbering sections to avoid conflicts (e.g., rename 3.8→3.7 before 3.7→3.6)
5. **For deleting sections**, close Find & Replace, select the entire section manually (click at start, shift-click at end), and press Backspace

## Boilerplate Sections (Do Not Modify)

These sections remain identical across all SOWs and should not be edited:
- Section 1: Purpose
- Section B: Full Marketing Operations Support (HubSpot)
- Section C: Marketing KPI Dashboards & Automation
- Section D: Documentation
- Section 3: Quality of Deliverables (all subsections)
- Section 4: Roles & Responsibilities (except resource count and names)
- Section 5: Confidentiality & Data Handling + Knowledge Transfer
- Exit Clause
- Acceptance Criteria
