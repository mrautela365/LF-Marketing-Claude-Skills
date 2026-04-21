# Connected App Inventory Template

Use this template to catalog all connected apps in the source sandbox before migration.

## How to Gather This Data

1. Navigate to: `https://app.hubspot.com/connected-apps/{sourcePortalId}/installed`
2. For each app in the list, document the fields below
3. For Private Apps, also visit: `https://app.hubspot.com/private-apps/{sourcePortalId}`
   and click into each app to capture its scopes

## App Inventory Table

For each app, capture:

| Field | Description | Example |
|-------|-------------|---------|
| App Name | Exact name as shown in Connected Apps | Example Integration App |
| Type | Marketplace or Private | Private |
| Install Date | When it was installed | YYYY-MM-DD |
| Installed By | Email of the person who installed it | installer@example.org |
| SF Related? | Does it depend on the Salesforce integration? | YES / NO |
| Still Needed? | Is this app actively used? | YES / NO / ASK OWNER |
| Migration Action | What to do in the target sandbox | Recreate / Reinstall / Skip |
| Priority | P0 (Critical), P1 (High), P2 (Medium), P3 (Low), Skip | P0 |
| External Credentials Needed? | Does reinstall require external service login? | Yes — SF OAuth |
| Owner for Credentials | Who can provide the external credentials? | Migration executor |
| Notes | Any additional context | Must be done first; all other SF apps depend on this |

## Classification Decision Tree

```
Is the app a test app (name contains "test", "CTJ-Test", etc.)?
  → YES: Skip — do not migrate
  → NO: Continue

Is the app a Marketplace app?
  → YES: Reinstall from HubSpot Marketplace
    Does it require OAuth with an external service?
      → YES: Need credentials owner identified
      → NO: Simple install, 2 minutes
  → NO: It's a Private App
    → Recreate from scratch in target sandbox
    Does an external system use this app's token?
      → YES: Must share new token with integration owner
      → NO: Just recreate with same scopes
```

## Private App Recreation Checklist

For each private app that needs to be recreated:

- [ ] Captured exact app name from source
- [ ] Captured all scopes from source (Settings → Private Apps → click app → Scopes tab)
- [ ] Created new private app in target with same name
- [ ] Set identical scopes
- [ ] Generated new access token
- [ ] Copied token immediately (shown only once!)
- [ ] Stored token securely
- [ ] Shared token with external integration owner (if applicable)
- [ ] External system updated with new token (confirmed)
- [ ] Verified app appears in Connected Apps in target sandbox

## Marketplace App Reinstall Checklist

For each marketplace app:

- [ ] Found app in HubSpot Marketplace
- [ ] Clicked Install and confirmed target portal
- [ ] Completed OAuth / authentication step (if required)
- [ ] Verified app appears as connected in target sandbox
- [ ] Tested basic functionality (if applicable)
