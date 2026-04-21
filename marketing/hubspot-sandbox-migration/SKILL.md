---
name: hubspot-sandbox-migration
description: >
  Plan, audit, and execute HubSpot sandbox migrations — moving configurations from a legacy/source
  sandbox to a new/target sandbox before a sunset deadline. Generates audit reports, step-by-step
  migration guides (XLSX), and tracks connected apps, Salesforce integrations, custom objects,
  field mappings, private apps, and workflows.
  
  Trigger on: HubSpot sandbox migration, legacy sandbox sunset, sandbox-to-sandbox migration,
  migrate HubSpot apps, recreate Salesforce integration in sandbox, custom object migration,
  sandbox deadline, migrate private apps, "move our sandbox", "new sandbox setup",
  "sandbox is being deleted", what transfers between sandboxes, sandbox permissions,
  how to recreate private apps, or any task involving moving HubSpot configs between portals.
---

# HubSpot Sandbox Migration

This skill helps you plan and execute a full migration from one HubSpot sandbox to another. It was
built from a real legacy → new sandbox migration and captures hard-won knowledge about what
transfers automatically, what doesn't, and exactly how to recreate everything by hand.

## When to Use This Skill

Use this skill when someone needs to:
- Migrate from a legacy HubSpot sandbox to a new one (especially before a sunset deadline)
- Audit what exists in a source sandbox (apps, integrations, custom objects, workflows)
- Generate a step-by-step migration guide for a team member to execute
- Understand what transfers automatically between HubSpot sandboxes and what doesn't
- Recreate Salesforce integrations, custom objects, and private apps in a new sandbox

## Critical Knowledge: What Transfers and What Doesn't

This is the single most important thing to understand. When HubSpot creates a new standard sandbox
from production, some things sync automatically and some things are completely absent:

### Transfers automatically (inherited from production at creation time)
- Contact properties (often hundreds, inherited from production)
- Workflows (often hundreds, though many may be unused)
- Standard HubSpot settings and configurations
- Lists and segments (structure, not necessarily data)

### Does NOT transfer — must be manually recreated
- **Connected apps** (marketplace AND private) — zero apps carry over
- **Salesforce integration** — must be completely reinstalled and re-authenticated
- **Custom object sync configurations** — field mappings and associations are NOT inherited
- **Private app tokens** — tokens are portal-specific and cannot be reused
- **Webhook URLs in workflows** that reference the old sandbox portal ID
- **Custom coded workflow actions** that use old private app tokens
- **OAuth connections** to external services (Salesforce, Demandbase, Slack, etc.)

This means the bulk of migration work is manually recreating integrations and apps.

## Migration Workflow

Follow these phases in order. Each phase has detailed guidance in the reference files.

### Phase 0: Discovery & Audit

Before migrating anything, you need a complete inventory of what exists in the source sandbox.

1. **Identify all connected apps** — Go to Connected Apps page in the source sandbox and document
   every app: name, type (Marketplace vs Private), install date, installer, and whether it's still
   needed.

2. **Audit the Salesforce integration** (if present):
   - Document the integration user (e.g., `hubspot@company.org.uat`)
   - For each synced standard object (Contacts, Companies, Deals, Tickets): capture all field
     mappings, sync direction, and conflict resolution settings
   - For each custom object: capture all field mappings, associations, sync direction, mapping type,
     and conflict resolution
   - Note the total field mapping count — this determines migration effort

3. **Audit workflows** — Look for workflows containing webhooks or custom coded actions. These will
   break in the new sandbox because they reference old portal IDs or old private app tokens.

4. **Check permissions** — Sandbox permissions are independent per portal. Being a Super Admin in
   production does NOT make you an admin in any sandbox. Verify who has admin access in both source
   and target sandboxes.

Read `references/audit-checklist.md` for the detailed audit checklist.

### Phase 1: Access & Prerequisites

Before any migration work:
- Ensure the executor has Super Admin access in BOTH source and target sandboxes
- Confirm Salesforce credentials are available for the integration user
- Verify the HubSpot managed package is installed in the Salesforce org (if SF integration is used)

### Phase 2: Salesforce Integration (the big one)

This is typically 60-80% of the migration effort. See `references/salesforce-migration.md` for
click-by-click instructions.

The process:
1. Install the Salesforce marketplace app in the target sandbox
2. Authenticate via OAuth with the SF org credentials
3. Configure standard object sync (Contacts, Companies, Deals, Tickets) — recreate all field
   mappings from the audit
4. Create custom object syncs — for each custom object, add every field mapping and association
   manually. This is the most time-consuming step. A single custom object can have 90+ field
   mappings.
5. Set conflict resolution to match the source (typically "Prefer Salesforce")
6. Validate sync health — check each object shows Active with no errors

### Phase 3: Marketplace Apps

Reinstall each marketplace app from the HubSpot Marketplace. Most require OAuth re-authentication
with the external service. Some require account credentials from the original installer.

See `references/app-inventory-template.md` for the inventory template.

### Phase 4: Private Apps

Private apps must be completely recreated — tokens don't transfer between sandboxes.

For each private app:
1. Create a new private app in the target sandbox with the same name
2. Copy the exact scopes from the source sandbox app
3. Generate the new access token — **copy it immediately** (you won't see it again)
4. Share the new token with whoever maintains the external integration
5. Update any workflows or external systems that used the old token

**Common mistake:** Forgetting to update external systems with the new token. The old token will
silently fail because it belongs to a different portal.

### Phase 5: Validate & Clean Up

1. Update workflows that reference old webhook URLs or tokens
2. Validate SF sync health for all objects
3. Test end-to-end: create a record in SF, verify it appears in HubSpot
4. Verify all apps show as connected
5. Delete the legacy sandbox only after full validation

## Generating the Migration Guide

When someone asks for a migration guide, generate an XLSX spreadsheet with these tabs:

1. **Migration Overview** — What, why, deadline, source/target portals, what's being migrated
   (list every app by name with type and priority), what transfers automatically vs. what doesn't

2. **Migration Steps** — Numbered steps grouped by phase, with columns:
   Step | Task | Owner | System | Detailed Instructions | Dependencies | Est. Time | Status
   
   The instructions column is critical — it should contain click-by-click detail:
   exact URLs (with portal IDs), exact button names, exact field values, exact dropdown selections.
   Think of it as a script that someone unfamiliar with HubSpot could follow.

3. **Field Mappings Reference** (if custom objects are involved) — Every field mapping for each
   custom object: SF field name, SF API name, HS field name, HS API name, sync direction, mapping
   type, conflict resolution. Include associations below the field list.

4. **Access Summary** — Who has what access in which portal, what's been verified, what still needs
   to be granted.

Build the XLSX using openpyxl (or your team's preferred spreadsheet library). The Migration Steps
tab benefits from color-coded phase headers, conditional formatting on the Status column, and
generous column widths for the Detailed Instructions column.

## Key Lessons Learned

These are patterns that emerged from real migration experience:

1. **Sandbox permissions are portal-independent.** This trips everyone up. You can be Super Admin
   in production and have zero access in a sandbox. Each portal has its own user permission set.
   There is no way to self-elevate — another admin must grant access.

2. **The production Sandboxes management page is limited.** It only offers "Delete" and "Set up a
   sync" — there is no user management from the production side.

3. **Custom objects auto-create when you set up sync.** You don't need to manually create the
   Membership or Project custom objects in the new sandbox. When you click "+ Sync custom object"
   and select the SF custom object, HubSpot creates the HS custom object automatically. But the
   field mappings and associations are NOT auto-configured — you must add each one manually.

4. **The 90-field mapping problem.** A single custom object can have 90+ field mappings. At ~30
   seconds per mapping, that's 45 minutes of repetitive clicking for ONE object. Budget 2-3 hours
   for large custom objects. Suggest the executor do this in one focused session with a second
   monitor showing the source sandbox for reference.

5. **Private app tokens are the silent killer.** Everything looks fine until an external system
   tries to call the HubSpot API with the old token. Map out every external system that uses each
   private app token and create a token distribution checklist.

6. **Workflows inherit from production but break silently.** The new sandbox gets hundreds of
   workflows from production, but any that contain webhooks pointing to the old sandbox or custom
   code using old tokens will fail. You need the workflow audit from Phase 0 to know which ones
   need updating.

7. **Always have someone with legacy admin access on the team.** If no one can see Workflows or
   Private Apps in the source sandbox, you can't audit what needs to be migrated. Get this access
   sorted out first.

8. **Document everything in the XLSX, not in Slack.** Migration guides get shared, re-read, and
   updated over days or weeks. A spreadsheet with all the detail in one place is far more useful
   than a Slack thread.

## Adapting This Skill

The patterns in this skill are universal. When adapting to a specific migration:

- Fill in portal IDs, usernames, and app names for the client
- The Salesforce integration workflow is identical regardless of organization
- Private app recreation is the same process everywhere
- The audit checklist applies to any HubSpot sandbox
- Field mapping counts will vary — scale the time estimates accordingly

## Reference Files

- `references/audit-checklist.md` — Complete audit checklist for source sandbox discovery
- `references/salesforce-migration.md` — Click-by-click Salesforce integration setup guide
- `references/app-inventory-template.md` — Template for cataloging connected apps
- `references/lf-migration-data.md` — Template for capturing migration-specific data (portal IDs,
  field mappings, app list) — fill in locally before running a migration
