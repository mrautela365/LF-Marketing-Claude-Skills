# Salesforce Integration Migration — Click-by-Click Guide

This guide covers reinstalling and configuring the HubSpot Salesforce integration in a new/target
sandbox. It assumes you've already completed the source sandbox audit and have all field mapping
data documented.

## Prerequisites

Before starting:
1. You have Super Admin access in the target HubSpot sandbox
2. You have valid Salesforce credentials for the integration user
3. The HubSpot managed package is installed in the Salesforce org
4. You have the complete field mapping audit from the source sandbox

## Step 1: Install the Salesforce Integration

1. Open: `https://app.hubspot.com/ecosystem/{targetPortalId}/marketplace/apps/sales/salesforce`
2. Click the orange "Install app" button
3. Confirm the portal name in the dialog matches your target sandbox
4. Click "Install app" again to confirm
5. HubSpot redirects to a Salesforce login screen
6. Enter the SF credentials:
   - Username: the integration user (e.g., `hubspot@company.org.uat`)
   - Password: the password for that user
7. Click "Log In"
8. Salesforce shows an "Allow Access" screen — click "Allow"
9. You're redirected back to HubSpot's SF integration settings

### Verification
On the "Overview" tab, confirm:
- Integration user shows the correct email
- Status shows "Connected" with a green checkmark

### Troubleshooting
| Issue | Cause | Fix |
|-------|-------|-----|
| "Disconnected" after auth | SF user lacks API access | Enable "API Enabled" permission in SF user profile |
| "Invalid credentials" | Wrong password or user | Reset password in SF Setup → Users |
| "Managed package not found" | Missing HubSpot package in SF | Install from SF AppExchange |
| OAuth redirect fails | Browser blocking popups | Allow popups for app.hubspot.com |

## Step 2: Configure Standard Object Sync

For each standard object (Contacts, Companies, Deals, Tickets), you need to set sync rules and
recreate field mappings.

### Sync Rules (per object)

1. Click the object's tab (e.g., "Contacts")
2. Click the "sync rules" sub-tab
3. Set sync direction to match source (typically "Sync both ways")
4. Set creation rules:
   - "When a contact is created in HubSpot..." → match source setting
   - "When a contact is created in Salesforce..." → match source setting
5. Set conflict resolution to match source (typically "Prefer Salesforce")

### Field Mappings (per object)

1. Click the "property mappings" sub-tab
2. For each mapping from your audit:
   a. Click "Add new mapping"
   b. Left dropdown: search for the Salesforce field name
   c. Right dropdown: search for the HubSpot field name
   d. Set the sync direction arrow (↔ for two-way, → or ← for one-way)
3. After mapping all fields, verify the total count matches your audit
4. Click Save if prompted

**Tip:** If the target sandbox inherited properties from production (which is typical), most HubSpot
fields should already exist. You're just creating the SF↔HS mapping link, not the fields themselves.

### Object-by-Object Navigation

| Object | URL Pattern |
|--------|-------------|
| Contacts | `https://app.hubspot.com/integrations-settings/{portalId}/installed/salesforce/contacts` |
| Companies | `https://app.hubspot.com/integrations-settings/{portalId}/installed/salesforce/companies` |
| Deals | Click "More" tab → Deals |
| Tickets | Click "More" tab → Tickets |
| Activities | Click "More" tab → Activities |

## Step 3: Configure Custom Object Sync

Custom objects are the most time-consuming part. Each must be added individually with all field
mappings created by hand.

### Adding a Custom Object

1. On the SF integration page, click "+ Sync custom object" (blue link, usually on the right)
2. A dropdown appears — select the custom object from Salesforce
3. HubSpot automatically creates a new custom object in HubSpot and opens its configuration page
4. You do NOT need to pre-create the custom object in HubSpot — it's created automatically

### Field Mappings

1. Click the "field mappings" sub-tab for the custom object
2. For each mapping from your audit:
   a. Click "Add new mapping"
   b. Left dropdown: search for the SF API name (e.g., `annual_price__c`)
   c. Right dropdown: search for the HS field name (e.g., "Annual Price")
   d. Set sync direction (typically "Two way")
3. Set conflict resolution to match source (typically "Prefer Salesforce")

**For large custom objects (50+ fields):**
- Open the source sandbox SF integration in a separate browser tab
- Work through the source field list page by page
- In the target tab, create each mapping to match
- Take breaks every 30 fields — this is tedious and errors from fatigue are real
- Budget 30 seconds per field mapping, so 90 fields ≈ 45 minutes

### Associations

After field mappings are done:
1. Click the "associations" sub-tab for the custom object
2. For each association from your audit:
   - Toggle it ON or OFF to match the source
   - If ON, the primary field and relationship type should auto-configure based on the SF schema
   - Set the sync direction to match source (e.g., "Salesforce to HubSpot only")
   - Set any association labels to match source

### Required Fields

Some custom objects have a "Required" mapping type (as opposed to "Custom"). These are typically
the object's primary identifier field (e.g., the auto-number or external ID). Make sure this
mapping is created — without it, sync will fail.

## Step 4: Validate Sync Health

After all objects are configured:

1. Go to the SF integration's "Sync Health" tab
2. Check each object:
   - Status should show "Active" with a green indicator
   - Error count should be 0
3. If errors appear:
   - Click the error count to see details
   - Common causes: unmapped required fields, data type mismatches, SF permission errors
   - Fix the mapping and recheck

### End-to-End Test

1. In Salesforce: create a test record (e.g., a new Contact)
2. Wait 10-15 minutes for sync
3. In HubSpot: search for that record
4. If it appears with correct field values, the sync is working
5. Repeat for at least one custom object record (e.g., create a test Membership in SF)

## Common Pitfalls

1. **Field name mismatches:** SF field names are case-sensitive in the API. `annual_price__c` is
   different from `Annual_Price__c`. Always use the exact API name from your audit.

2. **Picklist/enumeration mismatches:** If a SF picklist field maps to a HS enumeration, the
   picklist values must match. If they don't, records will sync but the field value may be blank.

3. **Currency fields:** SF currency fields may map to HS number fields. Check that the decimal
   precision matches.

4. **Date fields:** SF date vs. datetime can cause off-by-one-day issues due to timezone conversion.
   Verify a few test records after sync.

5. **Association junction objects:** Some associations use SF junction objects (e.g., Purchase History
   for Contact↔Membership). These junction objects must exist in the SF org with the correct lookup
   relationships. If the source sandbox was connected to the same SF org, they should already be there.
