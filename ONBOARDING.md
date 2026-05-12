# LF Marketing Ops — Onboarding Guide

This repo contains Claude Code skills for the Linux Foundation marketing team. Each skill is a reusable Claude workflow that automates a specific marketing task — from generating ad copy to syncing Slack requests into Asana.

---

## Prerequisites

Before using any skill, make sure you have:

- **Claude Code** installed and running ([claude.ai/code](https://claude.ai/code))
- Access to the relevant integrations for the skills you plan to use:

| Integration | Used by |
|---|---|
| Snowflake | Data queries across all analytics workflows |
| HubSpot | `hubspot-weekly-pulse`, `hubspot-sandbox-migration` |
| Asana | `slack-to-asana-pipeline`, `marketing-asana-setup`, `marketing-asana-task-update` |
| Slack | `slack-to-asana-pipeline`, `slack-workflow-intake` |
| Google Ads | `campaign-monitor`, `campaign-implementation` |
| Intercom | `intercom-banner-manager` |

---

## How Skills Work

Each skill lives in its own folder and contains a `SKILL.md` file. Claude reads this file when a skill is triggered and follows the instructions inside.

Skills are activated by **trigger phrases** — natural language descriptions in the `SKILL.md` frontmatter that tell Claude when to use that skill. You don't invoke skills by name; just describe what you want and Claude picks the right one.

**Example:**
> "Write me ad copy for our KubeCon sponsorship on LinkedIn and Google"

→ Claude automatically uses the `ad-copy-pro` skill.

Some skills also include an `assets/` subfolder with reference implementations (HTML tools, scripts) that Claude copies and customizes rather than building from scratch.

---

## Available Skills

### Marketing Skills (`marketing/`)

| Skill | What it does | Example triggers |
|---|---|---|
| `ad-copy-pro` | Generates ad copy for all paid media channels (Google, LinkedIn, Meta, Reddit, etc.) as a self-contained HTML tool | "Write paid media copy for our KubeCon event", "Build me an ad copy generator" |
| `campaign-brief-generator` | Creates structured campaign briefs | "Generate a campaign brief for our LFCA certification push" |
| `campaign-implementation` | Implements campaigns in ad platforms | "Set up the Google Ads campaign from this brief" |
| `campaign-monitor` | Browser-based monitoring of Google Ads performance | "Check how our current Google Ads campaigns are performing" |
| `communityseg-to-segment-migration` | Migrates audience data from CommunitySeg to Segment | "Migrate our community segments to Segment" |
| `hubspot-sandbox-migration` | Migrates HubSpot configuration between sandbox and production | "Copy this HubSpot workflow to production" |
| `hubspot-weekly-pulse` | Generates a weekly HubSpot performance report | "Pull the weekly HubSpot pulse", "What did our emails do this week?" |
| `intercom-banner-manager` | Manages banners and in-app messages in Intercom | "Update the Intercom banner for our upcoming event" |
| `lf-event-scraper` | Scrapes LF event data for use in campaigns | "Pull the upcoming events from the LF events page" |
| `lf-sow-generator` | Generates Statements of Work for LF campaigns | "Create an SOW for this sponsorship campaign" |
| `marketing-asana-task-update` | Updates Asana tasks with marketing campaign progress | "Update the Asana tasks for this week's campaigns" |
| `marketing-qa` | QA checks on marketing assets and copy | "QA this email before we send it" |

### Slack & Asana Skills (`slack-asana-skills/`)

| Skill | What it does | Example triggers |
|---|---|---|
| `slack-to-asana-pipeline` | Reads a Slack channel and creates Asana tasks from messages; posts a confirmation back to Slack | "Check my Slack for task requests", "Turn that Slack message into an Asana task" |
| `slack-workflow-intake` | Processes structured Slack workflow form submissions into Asana tasks | "Process the intake form submissions from Slack" |
| `marketing-asana-setup` | Sets up an Asana project structure for the marketing team | "Set up a new Asana marketing project" |

---

## Snowflake Setup

To query Snowflake data through Claude Code, follow the step-by-step instructions in [`snowflake-setup-guide.md`](./snowflake-setup-guide.md). It covers:

- Generating an RSA key pair for authentication
- Installing dependencies (`snowflake-sdk`, `dotenv`)
- Creating your `.env` file with connection credentials
- Adding the `snowflake-connection.js` helper module
- Testing your connection

Once set up, you can ask Claude things like:
> "Query Snowflake for all contacts where role is Technical Contact"

---

## Adding a New Skill

1. Create a folder under `marketing/` or `slack-asana-skills/` named after your skill (kebab-case)
2. Add a `SKILL.md` file with YAML frontmatter:
   ```yaml
   ---
   name: your-skill-name
   description: >
     One-paragraph description of what this skill does.
     Include trigger phrases — the natural language cues
     that should activate this skill.
   ---
   ```
3. Write the skill body: what Claude should do, step by step, with any constraints or examples
4. Optionally add an `assets/` subfolder for reference files (HTML tools, scripts, templates)

Look at `marketing/ad-copy-pro/SKILL.md` or `slack-asana-skills/slack-to-asana-pipeline/SKILL.md` for well-structured examples.
