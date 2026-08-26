# ISSUE Log - Zoho

## ISSUE:zoho 2026-08-26 -> Zoho People -- onboarding email alert Attachments field is single-file only

**Limitation:** the Attachments section on an onboarding email alert (Welcome Email to Candidate, etc.) holds only one file. Uploading a second file via Desktop or Cloud/Google Drive replaces the first rather than adding to it. The underlying `<input type="file">` has `multiple` set in the DOM, but the UI does not honour multi-select in practice, and doing three separate single-file "Upload from Desktop" actions in a row also just overwrites each time.

**Resolution:** zip the multiple files into one archive and attach the single .zip (a supported format). See ASSET:zoho 2026-08-26 -> Zoho People -- Welcome Email to Candidate template rebuilt (merged onboarding content).

## ISSUE:zoho 2026-08-26 -> Zoho People -- real onboarding email was manual via Outlook, not the Zoho template

**Finding:** the built-in "Welcome Email to Candidate" alert was built (see 2026Q2 log) but never actually used for real onboarding communication. In practice, HR sends the actual onboarding email (welcome message, start date, document checklist, device setup, contact info) manually via Outlook, with new hires emailing completed forms back -- the exact manual round-trip the platform consolidation was meant to eliminate. The Zoho template only ever sent a bare portal invite, disconnected from the real content.

**Resolution:** merged the Outlook email content into the Zoho template itself so one automated email covers both purposes -- see ASSET:zoho 2026-08-26 -> Zoho People -- Welcome Email to Candidate template rebuilt (merged onboarding content).
