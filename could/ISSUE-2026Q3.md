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

**Status 2026-09-16:** still unresolved -- the Workflow was enabled live to the real `servicedesk@transputec.com` anyway (see ASSET:zoho 2026-09-16 -> Zoho People -- notify-IT milestone shipped: scope simplified to device-name-only, no attachment needed), per explicit user decision to proceed without waiting on this. Worth confirming with IT/service-desk after the fact whether the generic subject line is auto-routing/categorizing correctly, since real candidate submissions will now generate real notifications with the unconfirmed subject format.

## ISSUE:zoho 2026-09-08 -> Zoho People -- Custom Function "Execute Script" test button fails with "Invalid Domain"

**Symptom:** clicking "Execute Script" (or "Save & Execute Script") on the `notifyIT_AttachScreenshot` Custom Function fails immediately with `Execution exception : Error due to - 'Invalid Domain'`.

**Isolation performed:** added temporary debug `return` statements to bisect which line throws. The error fires even on the very first built-in integration task in the script -- a bare `zoho.people.getRecords("Onboarding_Overseas_Staff",1,10)` call with no search criteria map at all -- before the code ever reaches the `invokeurl` file-download block or the OAuth connection. This rules out the `peoplefileaccess` connection, the search criteria map, and the file-download URL as the cause.

**Likely cause (not yet confirmed with Zoho support/docs):** a known-style limitation where the in-editor "Execute Script" test button runs the script in a sandbox context that cannot resolve the calling Zoho People domain (people.zoho.eu here) for built-in `zoho.people.*` integration tasks, even though the same call works when the function actually runs from a live trigger (Workflow, custom button, etc.) with real request context.

**Resolution (pending):** cannot use the test button to validate this function. Must instead test by wiring it into a real trigger and observing the live result (e.g., temporarily re-enabling the disabled "Notify IT" workflow, or attaching the function to a Custom Button and clicking it from an actual Onboarding Staff record) -- see ASSET:zoho 2026-09-08 -> Zoho People -- Custom Function built to attach the real screenshot file to the IT notification (IT has no Zoho access). Per [[feedback-zoho-confirm-before-live-actions]], this requires checking with the user before triggering, even against the test candidate.

**Resolved 2026-09-08:** confirmed via live testing (see ASSET:zoho 2026-09-08 -> Zoho People -- notifyIT_AttachScreenshot debugged and confirmed working live via a temporary Custom Button) that this is purely a test-button sandbox artifact -- the same `zoho.people.getRecords()` call works fine once the function is actually triggered from a real Custom Button click. Live triggering + the Workflow & Custom Button Logs page became the standard way to test/debug this function going forward.

## ISSUE:zoho 2026-09-08 -> Zoho People -- invalid searchField silently returns an error object instead of throwing

**Symptom:** `zoho.people.getRecords(formName, 1, 1, searchMap)` with `searchMap.put("searchField","Candidate")` did not error. `records.get(0)` returned an object that downstream code treated as a real record, but every `rec.get(...)` call on it silently returned null/empty -- no exception, just quietly wrong data.

**Cause:** `"Candidate"` is not a valid internal search field name for the Onboarding Staff form (the real name is `"CANDIDATE_ID"`, confirmed via a working `rec.get("CANDIDATE_ID")` elsewhere in the same script). The API's actual response to the bad field name is an error payload -- `{"code":7041,"message":"Field name 'Candidate' is invalid"}` -- but `zoho.people.getRecords()` returns this as if it were record 0 rather than raising an exception, so nothing in the calling script naturally detects the failure.

**How this was found:** dumped the raw `rec` map into a temporary debug email (`rec.toString()`) rather than trusting individual `.get()` calls, which is what actually revealed the error payload masquerading as data.

**Resolution:** always verify a search-based `getRecords()` result is a real record (e.g. check for an expected field, or that the map doesn't contain a `"code"`/`"message"` pair) before trusting it, especially when the search field name was typed by hand rather than picked from a verified list. Fixed here by correcting the search field to `"CANDIDATE_ID"`.

## ISSUE:zoho 2026-09-08 -> Zoho People -- file-upload field's `_downloadUrl` redirects to a different domain than the file bytes are served from

**Symptom:** using the file-upload field's companion `_downloadUrl` value (`https://people.zoho.eu/api/downloadFile?fcId=...&recordId=...&formId=...&original=true`) directly as the `invokeurl` target (with the `peoplefileaccess` connection) failed with `Execution exception : Error due to - 'Invalid Domain'`.

**Investigation:** captured the real network request the Zoho People UI itself makes when a user clicks to preview/download the same file -- it actually hits a completely different domain, `https://download.zoho.eu/webdownload?x-service=people&event-id=...&x-cli-msg={...}`, not `people.zoho.eu`. This suggested the `_downloadUrl` field's URL 302-redirects cross-domain to serve the real bytes, and that the OAuth connection (scoped to the Zoho People service/domain) was rejecting that redirect target.

**Wrong fix attempted:** fabricated a REST endpoint (`https://people.zoho.eu/api/v3/files/download?file_id=<fcId>`) guessing this would stay same-domain and avoid the redirect. This *did* avoid the "Invalid Domain" error, but the endpoint doesn't actually exist/return real file content -- `invokeurl` returned something that failed Deluge's own type check when treated as a file (`fileData.getFileName()` threw `Data type of the argument of the function 'getFileName' did not match the required data type of '[FILE]'`). The resulting email sent successfully (silently) with **no attachment**, since Deluge's `sendmail ... Attachments: file: fileData` appears to just drop an invalid/non-file attachment value rather than erroring the whole send.

**Actual resolution:** revert to using the field's real `_downloadUrl` value directly with `invokeurl` + connection -- this matches Zoho's own documented/community-example pattern for downloading file-upload-field content (`file.toCollection().get("download_Url")` + connection + `type: GET`). Re-tested live and the run logged clean `Successful` with no error. The original "Invalid Domain" failure on this exact same URL earlier in the session has not been fully explained (possibly a transient/flaky first-run issue since the connection had only just been created) -- flagged as unresolved but no longer blocking, since subsequent live runs using the same URL pattern succeeded without any code change to the URL construction itself.

**Still pending confirmation:** whether the actual email attachment is present and openable now that the correct download source is used again (checking in progress with the user at the time of this log entry).

## ISSUE:zoho 2026-09-08 -> Zoho People -- sendmail `from:` rejects unverified addresses with a misleading generic error

**Symptom:** setting `from: "hr@transputec.com"` in a Custom Function's `sendmail` block caused the **Save** button to fail with a generic `"The syntax is incorrect. Please check the format."` -- no indication of what was actually wrong. The specific reason only surfaced when using **"Save & Execute Script"** instead: `The "from" address can only be zoho.adminuserid, zoho.loginuserid, or a verified email address. Please specify a valid email address and try again.`

**Cause:** `hr@transputec.com` is not registered/verified as a sender address in this org's mail settings, and Deluge's `sendmail` restricts `from:` to the two built-in identity variables or a verified address.

**Resolution:** use `zoho.loginuserid` instead of a hardcoded unverified address -- resolves to whoever is actually logged in and triggering the action (confirmed via a live test: showed the real logged-in user's email, `jay.reck@transputec.com`, correctly). Note `zoho.adminuserid` (the other built-in option) does **not** behave the same way -- it resolves to the org's fixed configured admin/system account (here, "Roann Etan" / EMP798) regardless of who is actually logged in, which surprised the user since they were logged in as a different account. If a specific verified sender address (e.g. an HR shared mailbox) is wanted instead of either built-in, it must first be added/verified as a sender in Zoho Mail/People's admin settings -- not something a Deluge script can do on its own.

## ISSUE:zoho 2026-09-08 -> Zoho People -- Custom Function return value must be the literal string "success"

**Symptom:** a Custom Function triggered by a Workflow/Custom Button showed `Executed Successfully` as an on-screen toast, but the Workflow & Custom Button Logs page recorded the same run as `Failed`, with reason: `Response must contain 'success' return message. Function's response must contain 'success' (lowercase, spelled correctly) message to be considered successful. Any other response will cause failure. Return message: sent`.

**Cause:** the function's final `return "sent";` statement -- the script itself completed with no actual runtime error (the email had already sent by that point), but Zoho's Workflow/Custom Button success detection requires the literal return string `"success"`.

**Resolution:** always end a Custom Function meant to back a Workflow/Custom Button action with `return "success";` if it completed as intended -- any other return value marks the log entry `Failed` even when nothing actually went wrong, which is misleading when scanning the Logs page for real errors.

## ISSUE:zoho 2026-09-08 -> Zoho People -- Workflow & Custom Button Logs UI is unreliable for reading failure reasons

**Symptom:** hovering/clicking the colored status icon on a log row to see the `Reason:`/`Line number:` tooltip frequently showed nothing for the most recent row(s), even though the same technique worked instantly for older rows in the same list.

**Workaround found:** the tooltip text lives in the icon element's `title` attribute (`<i id="bp_info" class="PI_alert ...">`), but is only populated after a genuine, real (not synthetic/programmatic) click on that specific icon -- and even then can lag behind a live page. A full hard page reload (F5) followed by a fresh real click reliably worked when repeated in-page clicks did not. A separate "ⓘ" icon elsewhere in the row opens an unrelated "Execution details" / "Info messages" popup, which is not the same as the Reason tooltip and was frequently empty (`No info message available to display`).

**Also unreliable in the same UI:** the Custom Button's own confirm dialog ("A custom action to be trigger... click Confirm") only reliably appears on the very first click after a fresh page load -- repeated clicks against an already-settled page silently do nothing (no dialog, no new log entry, no error). Triggering the button via `document.querySelector(...).click()` in the browser console/devtools is more reliable than a coordinate-based UI click for automation purposes.
