# Migration Data Template

This file is a template for capturing the specifics of a HubSpot sandbox-to-sandbox migration.
Fill in the placeholders for your own migration. The original captured data has been removed
for public release; populate it locally before running the migration.

## Portal IDs

| Portal | ID | Role |
|--------|-----|------|
| Legacy Sandbox | `{legacySandboxPortalId}` | Source — being sunset on `{sunsetDate}` |
| New Sandbox | `{newSandboxPortalId}` | Target — created on `{createdDate}` by `{Migration executor}` |
| Production | `{prodPortalId}` | Production portal |

## Key Personnel

| Role | Notes |
|------|-------|
| Migration lead / Support | Super Admin in Production + new sandbox. Limited access in legacy sandbox. |
| Migration executor | Admin in legacy sandbox. Created the new sandbox. |
| Marketing Ops support | Available for questions, OAuth clicks, validation. |
| Private app owner(s) | Owner(s) of business-critical private apps that must be recreated. Share new tokens with them. |
| Former team member(s) | Original installer(s) of marketplace apps whose credentials may need to be recovered. |

## Salesforce Integration

- **Integration user:** `{integrationUser}` (the SF user account used by the legacy HubSpot ↔ SF connection)
- **Standard objects synced:** Contacts, Companies, Deals, Activities, Tickets (+ Overview)
- **Custom objects synced:** capture count and names from legacy
- **Conflict resolution:** capture from legacy (e.g., Prefer Salesforce)

## Connected Apps Inventory (Legacy Sandbox)

Capture all marketplace + private apps before the sunset deadline. Use this template:

| # | App Name | Type (Marketplace/Private) | Install Date | Installed By (role) | Action (Reinstall / Recreate / Skip) |
|---|----------|---------------------------|-------------|--------------------|--------------------------------------|
| 1 | Salesforce | Marketplace | | — | Reinstall + OAuth |
| 2 | (your apps) | | | | |

Mark test apps as SKIP and business-critical apps as HIGH priority.

## Custom Object Field Mappings

For each custom object, capture all field mappings. Example template structure:

| # | SF Field | SF API Name | HS Field | HS API Name | Type |
|---|----------|-------------|----------|-------------|------|
| 1 | | | | | |

Capture: sync direction, conflict resolution rule, REQUIRED vs Custom type, associations.

## Custom Object Associations

| Association | Primary Field | Relationship | Label | Sync Direction |
|-------------|--------------|--------------|-------|---------------|
| | | | | |

## Target Sandbox Starting State

Document at creation:

- Contact properties inherited from production
- Workflows inherited from production (note how many are active vs unused)
- Connected apps (typically 0 in a fresh sandbox)
- Custom objects (typically 0 in a fresh sandbox)

## Access Findings

| System | Migration lead's Role | Verified |
|--------|----------------------|----------|
| HubSpot Production (`{prodPortalId}`) | Super Admin | Yes — `{date}` |
| HubSpot New Sandbox (`{newSandboxPortalId}`) | Super Admin | Yes — `{date}` |
| HubSpot Legacy Sandbox (`{legacySandboxPortalId}`) | Limited (no workflows, no private apps) | Yes — `{date}` |

Key lesson: Production Super Admin does NOT grant sandbox access. Sandbox permissions are
portal-independent.

## Timeline Template

| Date | Event |
|------|-------|
| `{createdDate}` | Legacy sandbox created |
| `{newSandboxCreatedDate}` | New sandbox created |
| `{auditDate}` | Migration audit completed |
| `{guideDate}` | Migration guide generated |
| `{sunsetDate}` | Legacy sandbox sunset deadline |
