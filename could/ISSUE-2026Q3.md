# ISSUE Log - Zoho

## ISSUE:zoho 2026-08-26 -> Zoho People -- onboarding email alert Attachments field is single-file only

**Limitation:** the Attachments section on an onboarding email alert (Welcome Email to Candidate, etc.) holds only one file. Uploading a second file via Desktop or Cloud/Google Drive replaces the first rather than adding to it. The underlying `<input type="file">` has `multiple` set in the DOM, but the UI does not honour multi-select in practice, and doing three separate single-file "Upload from Desktop" actions in a row also just overwrites each time.

**Resolution:** zip the multiple files into one archive and attach the single .zip (a supported format). See ASSET:zoho 2026-08-26 -> Zoho People -- Welcome Email to Candidate template rebuilt (merged onboarding content).

## ISSUE:zoho 2026-08-26 -> Zoho People -- real onboarding email was manual via Outlook, not the Zoho template

**Finding:** the built-in "Welcome Email to Candidate" alert was built (see 2026Q2 log) but never actually used for real onboarding communication. In practice, HR sends the actual onboarding email (welcome message, start date, document checklist, device setup, contact info) manually via Outlook, with new hires emailing completed forms back -- the exact manual round-trip the platform consolidation was meant to eliminate. The Zoho template only ever sent a bare portal invite, disconnected from the real content.

**Resolution:** merged the Outlook email content into the Zoho template itself so one automated email covers both purposes -- see ASSET:zoho 2026-08-26 -> Zoho People -- Welcome Email to Candidate template rebuilt (merged onboarding content).

## ISSUE:zoho 2026-08-26 -> Zoho People -- email alert body silently truncates on save (emoji + large-paste corruption)

**Symptom:** saving an onboarding email alert body reports success ("Email alert updated successfully") but the actually-stored content can be silently truncated or corrupted. First case: body was cut off entirely right after a 😊 emoji character -- everything after it (multiple paragraphs, checklist, links) was gone on reload despite the success toast. Second case (separate, recurring across both automated typing and manual paste): long pasted text blocks dropped words/characters in place, particularly around `@` and long hyphenated ID strings (e.g. an Action1 agent URL's GUID segment got truncated to a few characters), and paragraph breaks around auto-linked URLs got silently eaten.

**Cause:** unclear -- likely an encoding issue with the emoji (4-byte UTF-8 / surrogate pair) breaking the save pipeline past that point, and a separate, unrelated paste-handling bug in the rich-text editor for long blocks of text. Both reproduced independent of whether the paste was done by a human or by browser automation.

**Resolution:** avoid emoji in template bodies entirely. After any large paste or save, don't trust the on-screen preview -- do a hard page reload and re-open the template fresh (both the read-only inline preview and the expanded iframe editor can show stale/cached content that doesn't match what's actually stored). Verify by reading the actual text and link hrefs, not just the visual render, then fix any corrupted spot with a small, targeted select-and-retype rather than another full paste.

## ISSUE:zoho 2026-08-26 -> Zoho People -- candidate creation rejects an email already tied to an org user

**Symptom:** Track Onboarding > Invite Candidate rejects an email with "The user is already part of current organization" if that address is already associated with an existing employee/user record in the org.

**Resolution:** use a genuinely external test address instead (not a company email or one already tied to a user account) when test-inviting a candidate to verify onboarding emails.

## ISSUE:zoho 2026-09-07 -> Zoho People/IT -- new device-notification workflow subject may not match existing ITSM ticket naming convention

**Finding:** this process already exists manually today, run outside Zoho People entirely. A real ITSM ticket (Servicely, `REQ0041028`, closed 2026-09-02) shows HR raises a ticket to service desk per new hire for Agent1 install + device name, titled with the convention:

`INT | Agent 1 Installation & Device Name - <Candidate Name> <date>`

(example: "INT | Agent 1 Installation & Device Name - Ara Yamit 12.10.2026" -- Ara Yamit is a real candidate, CND217, already in the live Track Onboarding list)

**Risk:** the new Workflow email alert built in ASSET:zoho 2026-09-07 -> Zoho People -- next milestone: auto-notify IT/service-desk on device name + screenshot submission currently uses a generic subject ("New Device Details Submitted for IT Setup"), not this convention. If Servicely auto-creates/categorizes/routes tickets from inbound email based on subject pattern (the "INT |" prefix in particular), a mismatched subject could land in the wrong queue or fail to auto-categorize -- can't verify this from inside Zoho People alone.

**Resolution (pending):** either update the workflow's email subject to match the "INT | Agent 1 Installation & Device Name - &lt;Name&gt; &lt;date&gt;" convention, or confirm directly with IT/service-desk whether subject-line matching actually matters for their ticket routing before relying on it.

## ISSUE:zoho 2026-09-08 -> Zoho People -- Custom Function "Execute Script" test button fails with "Invalid Domain"

**Symptom:** clicking "Execute Script" (or "Save & Execute Script") on the `notifyIT_AttachScreenshot` Custom Function fails immediately with `Execution exception : Error due to - 'Invalid Domain'`.

**Isolation performed:** added temporary debug `return` statements to bisect which line throws. The error fires even on the very first built-in integration task in the script -- a bare `zoho.people.getRecords("Onboarding_Overseas_Staff",1,10)` call with no search criteria map at all -- before the code ever reaches the `invokeurl` file-download block or the OAuth connection. This rules out the `peoplefileaccess` connection, the search criteria map, and the file-download URL as the cause.

**Likely cause (not yet confirmed with Zoho support/docs):** a known-style limitation where the in-editor "Execute Script" test button runs the script in a sandbox context that cannot resolve the calling Zoho People domain (people.zoho.eu here) for built-in `zoho.people.*` integration tasks, even though the same call works when the function actually runs from a live trigger (Workflow, custom button, etc.) with real request context.

**Resolution (pending):** cannot use the test button to validate this function. Must instead test by wiring it into a real trigger and observing the live result (e.g., temporarily re-enabling the disabled "Notify IT" workflow, or attaching the function to a Custom Button and clicking it from an actual Onboarding Staff record) -- see ASSET:zoho 2026-09-08 -> Zoho People -- Custom Function built to attach the real screenshot file to the IT notification (IT has no Zoho access). Per [[feedback-zoho-confirm-before-live-actions]], this requires checking with the user before triggering, even against the test candidate.
