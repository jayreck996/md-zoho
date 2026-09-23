# REQUEST Log - Zoho (External Dependencies / Infra Requests)

## REQUEST:zoho 2026-09-23 -> Azure AD -- App registration required for Zoho People SharePoint integration

**Requested from:** 3rd line support / M365 admin

**Context:** An automated Zoho People workflow (`saveFilesToSharePoint`) will push new hire documents uploaded on the Onboarding Staff form directly into a dedicated SharePoint folder per candidate. The workflow uses Microsoft Graph API, which requires a registered Azure AD app and a Zoho OAuth connection (`sharepointfileaccess`). The Zoho side is built and ready; only the Azure AD app and its credentials are blocking completion.

**What 3rd line support needs to do (one-off, ~10 minutes):**

1. **Register a new App Registration** in Azure Active Directory (portal.azure.com > Azure Active Directory > App Registrations > New Registration)
   - Name: `ZohoPeople-SharePoint`
   - Supported account types: Accounts in this organizational directory only (single tenant)

2. **Add a Redirect URI**
   - Platform: Web
   - URI: `https://deluge.zoho.eu/delugeauth/callback`

3. **Grant API Permissions** (API Permissions > Add a permission > Microsoft Graph > Delegated)
   - `Files.ReadWrite.All`
   - Click "Grant admin consent for [org]" after adding

4. **Create a Client Secret** (Certificates & Secrets > New Client Secret)
   - Description: e.g. `ZohoPeople integration`
   - Expiry: 24 months (or per org policy)
   - **Copy the secret Value immediately** -- it's only shown once

5. **Share back:**
   - **Tenant ID** -- Azure AD > Overview > Tenant ID (UUID)
   - **Client ID** -- App Registration > Overview > Application (client) ID (UUID)
   - **Client Secret value** -- the value from step 4

**What happens next (no further Azure access needed):**
Once the three values are received, the Zoho OAuth connection (`sharepointfileaccess`) will be created in Zoho People Developer Space, the Deluge function script will be saved, and the workflow will be enabled. All subsequent SharePoint access is handled by the app's delegated permissions -- no recurring admin involvement required.

**Status:** Pending -- email sent to 3rd line support 2026-09-23. Blocking: Custom Function `saveFilesToSharePoint` (ID: `5489000006525005`) + Workflow `Save Candidate Files to SharePoint`. See ASSET:zoho 2026-09-23 -> Zoho People -- saveFilesToSharePoint Custom Function created (stub, pending connection).
