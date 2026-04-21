# Source Sandbox Audit Checklist

Use this checklist before starting any migration. Every item must be documented before moving to
Phase 1.

## 1. Portal Identification

- [ ] Source sandbox portal ID
- [ ] Source sandbox name (displayed in top-right of HubSpot UI)
- [ ] Target sandbox portal ID
- [ ] Target sandbox name
- [ ] Production portal ID
- [ ] Migration deadline date

## 2. Access Verification

For each portal (source, target, production), verify:
- [ ] Who has Super Admin access?
- [ ] Does the migration executor have Super Admin in the source sandbox?
- [ ] Does the migration executor have Super Admin in the target sandbox?
- [ ] If not, who can grant it? (Must be an existing admin in that specific portal)

**How to check:** Navigate to `https://app.hubspot.com/settings/{portalId}/users` → search for the
user → click "Edit permissions" → look for the blue "Super Admins can access almost everything" box.

**Remember:** Production Super Admin does NOT carry over to sandboxes. Each portal has independent
permissions.

## 3. Connected Apps Inventory

Navigate to: `https://app.hubspot.com/connected-apps/{sourcePortalId}/installed`

For each connected app, document:
- [ ] App name
- [ ] Type: Marketplace or Private
- [ ] Install date
- [ ] Installed by (email)
- [ ] Still needed? (Yes / No / Ask owner)
- [ ] If Marketplace: will need OAuth re-auth with external service?
- [ ] If Private: who maintains the external integration that uses this token?

### Classification Guide

| Type | Migration Action | Effort |
|------|-----------------|--------|
| Marketplace (no external auth) | Install from Marketplace | 2 min |
| Marketplace (OAuth required) | Install + complete OAuth with external service | 5-10 min |
| Private (internal only) | Recreate app, copy scopes, generate new token | 5 min |
| Private (external integration) | Recreate + share new token with integration owner | 5 min + coordination |
| Test apps | Skip — do not migrate | 0 min |

## 4. Salesforce Integration Audit

If a Salesforce integration exists:

### Connection Details
- [ ] Integration user email (e.g., `hubspot@company.org.uat`)
- [ ] SF org type (Production, Sandbox, UAT)
- [ ] HubSpot managed package installed in SF org? Version?

### Standard Object Sync
For each synced standard object (Contacts, Companies, Deals, Tickets, Activities):

Navigate to: `https://app.hubspot.com/integrations-settings/{sourcePortalId}/installed/salesforce/{object}`

- [ ] Total field mapping count
- [ ] Sync direction settings (two-way, SF→HS only, HS→SF only)
- [ ] Conflict resolution setting (Prefer Salesforce, Prefer HubSpot, etc.)
- [ ] Any required/non-custom mappings flagged

### Custom Object Sync
For each custom object:
- [ ] Object name and HubSpot object ID
- [ ] Total field mapping count (this determines migration time — budget ~30 sec per mapping)
- [ ] All field mappings documented: SF field, SF API name, HS field, HS API name, sync direction,
      mapping type (Custom/Required), conflict resolution
- [ ] All associations documented: associated object, status (ON/OFF), primary field, relationship
      type, label, sync direction
- [ ] Estimated recreation time: (field count × 30 seconds) + 10 minutes for associations

### Capture Method
The best way to capture field mappings is to open the SF integration settings page and go through
each object's "property mappings" sub-tab. You can see this data even with read-only access — you
just can't edit it.

For very large custom objects (50+ fields), consider:
- Screenshot each page of mappings
- Have the executor work from two browser tabs: source in one, target in the other
- If you have API access, export via the HubSpot API

## 5. Workflow Audit

Navigate to: `https://app.hubspot.com/workflows/{sourcePortalId}`

**Important:** Workflow access requires admin permissions in that specific sandbox.

- [ ] Total workflow count
- [ ] Workflows with webhook actions — document: workflow name, webhook URL
- [ ] Workflows with custom coded actions — document: workflow name, code summary, any API tokens
      referenced
- [ ] Workflows referencing specific portal IDs in URLs — these need updating
- [ ] Workflows referencing private app tokens — these need new tokens after Phase 4

## 6. What the Target Sandbox Already Has

Check the target sandbox to understand your starting point:

- [ ] Contact property count (how many inherited from production?)
- [ ] Workflow count (how many inherited from production?)
- [ ] Any connected apps already installed?
- [ ] Any custom objects already configured?

Navigate to Settings → Properties, Settings → Workflows, and Connected Apps to check.

## 7. Stakeholder Map

- [ ] Migration executor: who will click the buttons? (Name, email, access level)
- [ ] Support team: who's available for questions? (Names, roles)
- [ ] SF admin: who can provide SF credentials or install managed packages?
- [ ] Private app owners: for each private app, who maintains the external integration?
- [ ] Approval authority: who signs off on deleting the legacy sandbox?

## Output

Package all of this into a migration guide (XLSX format recommended). See the SKILL.md for the
recommended tab structure.
