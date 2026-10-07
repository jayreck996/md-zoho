# ASSET Log - Zoho

## ASSET:zoho 2026-08-26 -> Zoho People -- platform decision: consolidate onboarding onto Zoho People (single platform)

- Decision: move new-hire onboarding off the stitched-together Learn + Forms + Flow + WorkDrive setup (see 2026Q2 log) onto Zoho People, aiming for one seamless platform instead of automation glue across separate apps
- Rationale: prior setup existed as a workaround for gaps in Learn/Forms/Flow (no Zoho Mail connection, no Learn enrollment trigger, no native attachments, no shared sandbox) -- People has onboarding natively (docs, e-signature, forms, checklists) so should close most of those gaps directly
- Scope to confirm during setup: whether Learn is retained for training content with People handing off to it, or whether People absorbs that too
- Status: decided -- setup in progress

## ASSET:zoho 2026-08-26 -> Zoho People -- email template location and native attachment support

- Location: Settings (gear icon) > Onboarding > Automation > Notifications -- lists all onboarding email alerts (Welcome Email to Candidate, Welcome Email to Employee, Candidate Onboarding Reinitiated Email, Employee Onboarding Reinitiated Email) plus a separate Onboarding Reminder email, each with its own toggle and "Edit Template" button
- Each template's editor (Edit Email Alert) has a native **Attachments** section directly under the message body, with three sources: Desktop (direct upload), Cloud (WorkDrive/cloud storage), Select Templates (pre-saved attachment library)
- Supported formats: .CSV, .DOC, .PDF, .XLS, .XLSX, .ZIP, .JPG, .JPEG, .PNG
- Attach once on the template and it auto-sends with every email that alert fires -- no separate Flow step or WorkDrive share-link workaround needed
- Confirmed on portal: transputeconboarding (people.zoho.eu), logged in as Super Administrator
- Supersedes: ISSUE:zoho 2026-06-23 -> Zoho Learn -- native email notifications do not support file attachments (2026Q2 log) -- that gap does not exist in Zoho People's onboarding email alerts
- **Correction (same day):** the Attachments section only holds ONE file at a time -- each Desktop/Cloud upload replaces the previous attachment rather than adding to it. Confirmed both via Desktop upload and via Cloud/Google Drive. The underlying file input has `multiple` set in the DOM, but the UI does not honour multi-select in practice.
- Workaround: zip multiple files into a single archive (.ZIP is a supported format) and attach that one file -- see ASSET:zoho 2026-08-26 -> Zoho People -- Welcome Email to Candidate template rebuilt (merged onboarding content)

## ASSET:zoho 2026-08-26 -> Zoho People -- Welcome Email to Candidate template rebuilt (merged onboarding content)

- Context: the built-in "Welcome Email to Candidate" alert (bare portal-invite email) was never actually used in practice -- HR has been manually sending the real onboarding email via Outlook instead (welcome message, start date, document checklist, device setup instructions, contact info), with a separate, disconnected Zoho portal invite. Two uncoordinated emails, one of them entirely manual.
- Rebuilt the template body to merge both: kept Zoho's native mechanics (password-setup link, portal-login link, both live merge-token hrefs `${CandidateInvitationURL}` / `${CandidateLoginURL}`) and folded in the full Outlook email content -- welcome message, `${Tentative_Joining_Date}` (only date field available, no separate start-time field on the candidate record), document checklist (CV, Proof of ID, Utility Bills, References, Police Clearance for overseas contractors, Speed Test, the 3 attached forms), Device Setup section with Action1 agent install links (Windows/Mac), and HR contact (hr@transputec.com, auto-linked as mailto).
- Subject changed from bare "Welcome Candidate" to "Welcome to ${companyName}, ${First_Name} - Onboarding Requirements".
- Attached `Onboarding-Forms.zip` (Data Protection Act Form, Personnel Questionnaire, Reference Request Consent Form) per the single-attachment-slot limitation above.
- Build note: the rich-text editor lives in an iframe reached only via "Edit Email Template" (the inline body preview in the alert modal is read-only and silently no-ops on typed input) -- edits must go through that expanded editor. Large single-paste blocks of the full merged body dropped words/characters in several places (particularly around `@` and long hyphenated strings) and silently ate paragraph breaks around auto-linked URLs; verified and fixed by reading `iframe.contentDocument.body.innerText` directly rather than trusting the visual render, which itself does not always match live state either.
- Status: built, verified (subject + full body text + all 5 links + attachment), and saved live -- "Email alert updated successfully" confirmed 2026-08-26.
- **Correction: the "verified and saved live" status above was premature.** A full page reload after that save showed the body silently truncated server-side, cutting off right where a 😊 emoji had been typed -- everything after it (checklist, device setup, links, closing) was gone from the live template despite the success toast. See ISSUE:zoho 2026-08-26 -> Zoho People -- email alert body silently truncates on save (emoji + large-paste corruption). Rebuilt without the emoji, re-verified via reload + DOM read (not the on-screen preview) -- confirmed complete and correct as of the final check.

## ASSET:zoho 2026-08-26 -> Zoho People -- no workflows attached to Onboarding module (confirmed before test trigger)

- Before running a real test invite, checked Settings > Onboarding > Automation > Workflows > "Manage Workflows" (Form: All) -- list is completely empty
- Confirms triggering a candidate's onboarding (the "Initiate associated workflows?" prompt in the Invite Candidate wizard, step 3) has nothing to actually initiate -- no hidden automation cascade tied to candidate creation/trigger in this org
- Relevant when judging blast radius of any test candidate/trigger: only the Welcome Email itself fires, nothing else in Zoho People or connected systems

## ASSET:zoho 2026-08-26 -> Zoho People -- test candidate created for live send verification (paused before trigger)

- Attempted test candidate with jayreck996@gmail.com via Track Onboarding > Invite Candidate -- blocked: "The user is already part of current organization" (that address is already a known user/employee record in this Zoho People org)
- Created test candidate instead with jay.reck@icloud.com: CND216, "Test Onboarding", Location New Zealand, Title Support Manager (auto-populated defaults, not set intentionally) -- status "Not Triggered"
- Reached step 3 of the Invite Candidate wizard (Trigger Onboarding: Yes, Initiate associated workflows: Yes) -- **paused before clicking Finish** at user's request, to trigger later same day
- Paused pending HR admin sign-off, since this is the live production template (Zoho People has no separate sandbox/test copy -- triggering it fires the exact same template real candidates receive)
- Sign-off received 2026-09-02. Test candidate CND216 (jay.reck@icloud.com) left untriggered; instead created a second test candidate CND219 (Jay Reck, ync5389@gmail.com) and triggered onboarding for real
- **Confirmed working end-to-end:** email received at ync5389@gmail.com, user confirmed "it works" -- subject, body, links, and zip attachment all functioning as built in the live template
- Status: production template verified via real send. CND216 and CND219 are test records still sitting in the live candidate list (Not Triggered / Triggered) -- consider deleting them once no longer needed to keep the real candidate list clean

## ASSET:zoho 2026-08-27 -> Tooling -- claude-in-chrome browser extension/MCP (used for this Zoho work)

- Not a Zoho product -- documented here because this is the tool actually used to configure the Zoho People onboarding email template in this quarter's entries above, for anyone reproducing or continuing this work
- What it is: an Anthropic-provided Chrome extension that connects the browser to a Claude session via MCP tools, letting Claude view and drive a real Chrome tab -- screenshots, clicks, typing, reading the page's DOM/accessibility tree, running JS -- instead of an invisible/headless browser
- Why it was needed here: this environment's default browser-automation path (a separate "wmux browser" panel) wasn't rendering Zoho People's page content correctly -- it kept returning the outer app shell instead of the actual page. Switched to claude-in-chrome and it worked reliably for the rest of the People configuration work.
- Session/permission model: Claude opens and controls tabs inside a dedicated tab group scoped to that session -- not arbitrary access to all open browser tabs/windows. Claude does not see or handle login credentials -- the user logs in manually in that tab; Claude only interacts with the page afterward.
- Tools available (MCP), as used this session: `tabs_context_mcp` (list/create the session's tab group), `navigate`, `computer` (mouse/keyboard/screenshot), `read_page` / `find` (accessibility tree), `javascript_tool` (run JS in page context -- used repeatedly here to verify what was actually saved server-side vs. what the on-screen preview showed), `file_upload` (attach local files to a file input, bypassing the native OS file picker), `resize_window`, plus `read_console_messages` / `read_network_requests` / `gif_creator` (not used this session)
- Install/setup: install the Claude Chrome extension from the Chrome Web Store, sign in, then grant it permission for the sites Claude should access. Exact install flow and availability can change over time -- check Anthropic's current documentation rather than relying solely on this note when setting it up fresh.

## ASSET:zoho 2026-09-02 -> Zoho People -- original "Welcome Email to Candidate" content preserved (no native version history)

- The Edit Email Alert editor has no revision history / "restore previous version" feature -- no undo back to a prior save once a template has been edited and saved
- Preserved here as the manual rollback point, since the rebuilt template (see ASSET:zoho 2026-08-26 -> Zoho People -- Welcome Email to Candidate template rebuilt) replaced this entirely and is now confirmed live in production:
  - **Original Subject:** "Welcome Candidate"
  - **Original Body:** "Hi ${First_Name},\n\nGreetings from ${companyName}!\n\nClick here to set up your password to login to our portal.\n\nOnce you have set up your account, you can complete onboarding process.\n\nTo login to the portal again, please use this link.\n\nRegards,\n${Person performing this action}" ("here" links to `${CandidateInvitationURL}`, "link" links to `${CandidateLoginURL}`)
  - No attachments, no document checklist, no device setup section on the original version
- If reverting is ever needed, this content plus the two live merge-token links above is everything required to manually rebuild it

## ASSET:zoho 2026-09-07 -> Zoho People -- next milestone: auto-notify IT/service-desk on device name + screenshot submission

- Goal: when a new hire submits their computer/device name and a screenshot, automatically email IT/service-desk instead of relying on someone noticing it manually
- Checked the **Candidate** form (Settings > Onboarding > Extend Service > Forms > Candidate) first -- it has no device-name or screenshot field. The onboarding checklist email tells candidates to submit "Speed Test Screenshot" and "Machine/Device Name" "via the portal," but no such fields exist on that form -- those checklist lines were text only, with nothing behind them on the Candidate form.
- Checked the separate **Onboarding Staff** form instead (same Extend Service > Forms list, form id 5489000001985025, label `Onboarding_Overseas_Staff`) -- it already has both fields needed, plus everything else on the checklist:
  - **Machine/Device Name** (If applicable write Company Issued Laptop) -- required single-line text
  - **Upload Screenshot of Machine/Device Name** -- required Desktop/Cloud upload
  - Also present: Home Address, Personal Email Address, Job Role Offered, Reference 1/2 Email, Police Clearance Certificate (PCC -- remote workers only), Proof of Identity, Screenshot from speedtest.net, CV, 2nd Utility Bill, Data Protection Act Form, Referencing Consent Form, P45/HMRC Checklist (UK staff only), Next of Kin Name & Contact Number
- Conclusion: **no new form fields need to be built** -- the milestone is purely an automation. Build a Workflow (Settings > Onboarding > Automation > Workflows -- confirmed empty in the earlier ASSET entry above) that fires an email to IT/service-desk when this Onboarding Staff record has Machine/Device Name + the device screenshot populated.
- Open questions before building, asked of user:
  1. Which email address(es) should receive the notification (IT/service-desk distro vs. a specific person)?
  2. Trigger the moment either field is filled, or only once both are filled together (avoid a half-complete notification)?
- Answers: both fields required together; recipient servicedesk@transputec.com
- **Built:** new Workflow "Notify IT - Device Name and Screenshot Submitted" on the Onboarding Staff form (Settings > Onboarding > Automation > Workflows):
  - Trigger: Existing record is edited -> Execute only once (fires the first time the criteria matches, not on every subsequent edit)
  - Criteria: `(1 AND 2)` -- Machine/Device Name Is Not Empty AND Upload Screenshot of Machine/Device Name Is Not Empty
  - Action: Email Alert "Notify IT - Device Details Submitted" to servicedesk@transputec.com, subject "New Device Details Submitted for IT Setup", body includes candidate name (via `${CANDIDATE_ID.First_Name}` / `.Last_Name}` -- lookup-field dot-notation, found under the separate "Candidate (Candidate)" category in the merge-field picker, since the plain "Candidate" field under Onboarding Staff only inserts the raw `${CANDIDATE_ID}` record ID), Job Role Offered, and the device name value, plus a note that the screenshot itself is attached to the Onboarding Staff record (not re-attached to this notification email)
- Left the pre-existing, unrelated "Onboarding Overseas Staff" workflow draft (Create trigger, no criteria, no actions) untouched per user's choice to build a new, separate workflow rather than repurpose it
- Status: built and saved live 2026-09-07, then **disabled** 2026-09-08 at user's request pending resolution of the ITSM subject-line question below (see ISSUE:zoho 2026-09-07 -> Zoho People/IT -- new device-notification workflow subject may not match existing ITSM ticket naming convention) -- not yet tested end-to-end with a real record edit
- Note on toggling this workflow: the Status switch's `aria-checked` attribute is unreliable (shows "false" even on workflows that are actually on) -- go by the visible toggle color/position (blue = enabled, grey = disabled) instead. Disabling prompts a confirmation dialog ("Disable workflow? Workflow execution will be stopped. This action will not impact previous records.") that must be confirmed.

## ASSET:zoho 2026-09-08 -> Zoho People -- Custom Function built to attach the real screenshot file to the IT notification (IT has no Zoho access)

- Hard constraint from user: **IT has no access to Zoho People at all**, so a notification that just says "go check the record in Zoho" is unacceptable -- the actual uploaded screenshot file must be delivered as a real email attachment.
- The no-code Email Alert action (used in the workflow above) cannot do this: its Attachments field only supports a static, pre-chosen file (Desktop/Cloud upload), not a per-record dynamically-uploaded file. Confirmed there is no field-picker option to insert an uploaded file as a dynamic attachment token.
- Built a Deluge **Custom Function** `notifyIT_AttachScreenshot` (Settings > Onboarding > Automation > Actions > Custom Functions, Onboarding Staff form) to do this via the Zoho People REST API instead:
  - Parameter: `candidateId` (string) mapped via Edit Parameters to the **"Candidate (Onboarding Staff)"** lookup category (not the plain "Candidate" field, which only yields the raw `${CANDIDATE_ID}` record ID)
  - Script: looks up the Onboarding Staff record via `zoho.people.getRecords("Onboarding_Overseas_Staff",1,1,searchMap)` searching by Candidate = candidateId; reads `machine_device_name` and the file-upload field's companion `_downloadUrl` field (`upload_screenshot_of_machine_device_name_downloadUrl`); downloads the actual file via `invokeurl` (GET, using the OAuth connection below) into `fileData`; sends the notification via `sendmail` with `Attachments: file: fileData`
  - Requires an OAuth **Connection** (Settings > Developer Space > Connections, or via the function editor's own Connections tab) named `peoplefileaccess` ("PeopleFileAccess"), service Zoho People, scope `ZOHOPEOPLE.files.ALL`, status Connected -- created and authorized this session
  - Test config used per user's explicit instruction: recipient `jayreck996@gmail.com` (not the real servicedesk@transputec.com) and test record CND219 (Jay Reck, ync5389@gmail.com, candidate record id `5489000006434011`), which the user confirmed has a real uploaded screenshot
- Saved successfully (2026-09-08). See ISSUE:zoho 2026-09-08 -> Zoho People -- Custom Function "Execute Script" test button fails with "Invalid Domain" for the blocker hit when trying to test it via the in-editor test button, and why that likely does not affect a real, live-triggered execution.
- **Not yet wired into the "Notify IT" Workflow** (still disabled from the entry above) and not yet verified end-to-end with a real send -- next step is to test via an actual trigger (e.g., re-enabling the workflow and editing the real CND219 Onboarding Staff record, or a Field Update-based custom button) rather than the broken test button, pending user confirmation per [[feedback-zoho-confirm-before-live-actions]].

## ASSET:zoho 2026-09-08 -> Zoho People -- notifyIT_AttachScreenshot debugged and confirmed working live via a temporary Custom Button

- Built a temporary **Custom Button** "TEST Notify IT (Attach Screenshot)" (Onboarding > Extend Service > Custom Button, Onboarding Staff form, positioned "In Record View") mapped to the `notifyIT_AttachScreenshot` custom function as its default action. This became the reliable way to live-test the function against a real record on demand, since:
  - The in-editor **"Execute Script" test button is unreliable for any `zoho.people.*` built-in call** -- it fails with `Invalid Domain` even on a bare `zoho.people.getRecords()` with no search criteria, isolated via bisection with temporary debug `return` statements. This looks like a sandbox limitation of the test tool itself (can't resolve the calling People domain), not a real script defect -- the same call works fine once triggered live.
  - The **Onboarding Staff module's own list view** (Operations > Onboarding > Onboarding Staff, and the Settings-side equivalent) shows "No records found" for real existing records regardless of the "All Data" filter -- a UI-only display bug (confirmed the data exists via direct API/Deluge calls). To reach a specific Onboarding Staff record's actual UI page (needed to place/click the Custom Button), go via **Operations > Onboarding > Track Onboarding > Onboarding in Progress > click the candidate's task-completion status (e.g. "3/3 In progress") > click the "Onboarding Staff" link** in the resulting Onboarding Tasks popup -- this opens the real record and reveals its true internal `recordId` (distinct from the Candidate record's own id) in the URL.
  - The button's confirm dialog only reliably fires on a **freshly loaded page** (F5 hard reload, or first render after in-app navigation) -- repeated clicks on an already-settled page silently do nothing (no dialog, no log entry). Triggering the button via `document.querySelector(...).click()` in JS is more reliable than coordinate-based clicks, since the on-screen viewport/screenshot scale factor drifted unpredictably mid-session (screenshots ranged from 959px to 1186px wide against a constant 1307px `window.innerWidth`), causing visually-correct-looking clicks to land on the wrong element.
  - The **Workflow & Custom Button Logs** page (Settings > Onboarding > Automation) is the authoritative result -- the on-screen "Executed Successfully" toast is not trustworthy on its own (it fired on runs that the log then recorded as `Failed`). Each log row's colored status icon carries the real reason in its `title` attribute, populated only after a live (not synthetic) click on it; a second, separate "ⓘ" icon in the row opens an "Execution details" popup with the raw arguments/info messages.
- **Two real bugs found and fixed through iterative live-testing** (isolated by inserting temporary debug `return`/`sendmail` statements dumping intermediate variable values -- since the test-button sandbox couldn't be used, all diagnosis had to happen via real triggered runs read back through the Logs page):
  1. **`searchField: "Candidate"` was an invalid API field name** -- `zoho.people.getRecords()` didn't throw on this; it silently returned an error object (`{"code":7041,"message":"Field name 'Candidate' is invalid"}`) as if it were record 0, so every downstream `rec.get(...)` call returned null/empty with no indication the record itself was garbage. Fixed by changing the search field to `"CANDIDATE_ID"`, matching the field's real internal name (confirmed via the working `rec.get("CANDIDATE_ID")` used elsewhere in the same script).
  2. **The file download URL construction was wrong.** The field's companion `_downloadUrl` value (e.g. `https://people.zoho.eu/api/downloadFile?fcId=...`) redirects to a *different* domain (`download.zoho.eu`) to actually serve the bytes -- confirmed by capturing the real network request the browser itself makes when previewing the file. An attempt to work around this by fabricating a `people.zoho.eu/api/v3/files/download?file_id=...` REST endpoint was a **wrong guess**: it didn't error, but also didn't return real file bytes (confirmed via `fileData.getFileName()` throwing `Data type of the argument ... did not match ... [FILE]`), so the resulting email sent successfully with no attachment. **Fix:** revert to using the field's own `_downloadUrl` value directly with `invokeurl` + the `peoplefileaccess` connection -- this matches Zoho's own documented/community pattern (`file.toCollection().get("download_Url")` + connection + GET) and is the correct approach; the domain-redirect theory turned out not to be the real blocker.
  - Also found: **Deluge's `sendmail` `from:` field only accepts `zoho.adminuserid`, `zoho.loginuserid`, or a pre-verified sender email** -- attempting `from: "hr@transputec.com"` (not a verified sender in this org) was rejected, but surfaced only as a generic, misleading `"The syntax is incorrect. Please check the format."` on the plain Save button (the specific reason only appeared via "Save & Execute Script"). `zoho.adminuserid` resolves to the org's configured system/admin account (in this org, "Roann Etan" / EMP798) regardless of who is actually logged in and clicking -- not the current session's identity. Switched to `zoho.loginuserid`, which correctly resolves to whoever is actually triggering the action (confirmed: showed `jay.reck@transputec.com`, the real logged-in session, once switched).
  - Also found: a Custom Function's return value must be the **literal string `"success"`** for a Workflow/Custom Button action to record it as `Successful` -- returning `"sent"` (or anything else) marks the run `Failed` in the Logs even though the script completed with no actual error (`Reason: Response must contain 'success' return message ... Return message: sent`). Fixed by changing the final `return` statement to `"success"`.
- **Current state of `notifyIT_AttachScreenshot`:** uses `searchField:"CANDIDATE_ID"`, downloads via the field's real `_downloadUrl` + `peoplefileaccess` connection, sends `from: zoho.loginuserid`, and `return "success";`. Live-tested via the Custom Button against CND219 and logged clean `Successful` with no error reason on the two most recent runs -- **whether the email attachment itself is actually present is still pending final confirmation from the user** (checking jayreck996@gmail.com); a prior run using the wrong v3-endpoint URL confirmed the email arrives with the correct subject/body but silently missing the attachment when `fileData` isn't a real file object, so this must be re-verified now that the download source is fixed.
- Testing paused here at user's request ("park the testing") pending that final email/attachment confirmation. The temporary "TEST Notify IT (Attach Screenshot)" Custom Button is still live on the Onboarding Staff form and should be removed (or left, if useful for future testing) once the flow is fully confirmed and wired into the real disabled Workflow.

## ASSET:zoho 2026-09-16 -> Zoho People -- notify-IT milestone shipped: scope simplified to device-name-only, no attachment needed

- **Requirement clarified by user:** the real-world purpose of the servicedesk notification is for IT to confirm the "Agent1" remote-management agent got installed on the candidate's named device -- it only needs the device name/title, not a screenshot of it. This removes the entire reason the `notifyIT_AttachScreenshot` Custom Function (file download, OAuth connection, attachment) was built in the first place.
- **Simplified `notifyIT_AttachScreenshot`** (Settings > Onboarding > Automation > Actions > Custom Functions, Onboarding Staff form) down to just: look up the record by `CANDIDATE_ID`, read `machine_device_name` and `CANDIDATE_ID` (candidate name), `sendmail` with `from: zoho.loginuserid` and a plain-text body (candidate name + device name, no attachment), `return "success";`. Removed the `fileUrl`/`_downloadUrl` lookup, the `invokeurl` download block, the `peoplefileaccess` connection dependency, and the `Attachments:` line entirely.
- Live-tested via the "TEST Notify IT (Attach Screenshot)" Custom Button against CND219 -- logged clean `Successful` in Workflow & Custom Button Logs, and the user confirmed the resulting email (subject "New Device Details Submitted for IT Setup - CND219 Jay Reck", to jayreck996@gmail.com) arrived correctly with just candidate name + device name.
- **Enabled the "Notify IT - Device Name and Screenshot Submitted" Workflow** (previously disabled, see the 2026-09-07/08 entries above) -- this Workflow uses its own separate no-code **Email Alert** action ("Notify IT - Device Details Submitted"), not the Custom Function, since the Custom Function's only advantage (dynamic attachment) is no longer needed:
  - **Criteria simplified:** removed the `Upload Screenshot of Machine/Device Name Is Not Empty` condition -- now fires on `Machine/Device Name Is Not Empty` alone (per user decision), so it no longer waits on the screenshot field at all.
  - **Recipient confirmed as the real `servicedesk@transputec.com`** (per user decision) -- this Workflow is now live in production for real candidates going forward, not just test records.
  - Status toggled on (blue) and saved successfully.
- **Not yet resolved / left open:** the workflow name "Notify IT - Device Name and **Screenshot** Submitted" is now slightly stale wording given the criteria no longer checks for the screenshot -- a cosmetic rename was not requested/made. The ITSM subject-line naming-convention question (see ISSUE:zoho 2026-09-07 -> Zoho People/IT -- new device-notification workflow subject may not match existing ITSM ticket naming convention) also remains unresolved and was not blocking this go-live per user's explicit decision to proceed with the real recipient anyway.
- The `notifyIT_AttachScreenshot` Custom Function and its "TEST Notify IT (Attach Screenshot)" Custom Button are no longer part of the live path (the enabled Workflow uses the plain Email Alert instead) -- both can be left as-is for potential future use or removed as cleanup; not actioned either way this session.

## ASSET:zoho 2026-09-21 -> Zoho People -- next milestone: auto-save all candidate files to SharePoint on form submission

- **Goal:** when a candidate submits the Onboarding Staff form, every uploaded file (CV, Proof of ID, Police Clearance, Utility Bill, Speed Test screenshot, Device screenshot, Data Protection Form, Referencing Consent, P45/HMRC Checklist) is automatically pushed into a dedicated per-candidate folder in SharePoint -- no manual retrieval from Zoho People needed.
- **Approach:** a new Deluge Custom Function `saveFilesToSharePoint` triggered by a new Workflow on the Onboarding Staff form (trigger: New record is added, execute only once). Completely separate from `notifyIT_AttachScreenshot` -- same download mechanics (`_downloadUrl` + `peoplefileaccess` connection + `invokeurl`), different destination and scope.
- **SharePoint folder structure:** `/HR/Onboarding/{CandidateID} - {First_Name} {Last_Name}/` (e.g. `/HR/Onboarding/CND219 - Jay Reck/`), with each file saved under its original filename.
- **Pre-conditions before building (must be done first):**
  1. **Azure AD App Registration** -- an OAuth app is required in the M365 tenant to call Microsoft Graph API. Register at Azure Portal > Azure Active Directory > App Registrations > New Registration (name: `ZohoPeople-SharePoint`). Redirect URI: Zoho's OAuth callback as shown in the Connections setup screen. API Permission: `Sites.ReadWrite.All` (or `Files.ReadWrite.All`). Create a Client Secret; note Client ID + Secret + Tenant ID.
  2. **Zoho Connection `sharepointfileaccess`** -- Settings > Developer Space > Connections > New Connection, Custom OAuth / Microsoft. Authorization URL: `https://login.microsoftonline.com/{tenantId}/oauth2/v2.0/authorize`; Token URL: `https://login.microsoftonline.com/{tenantId}/oauth2/v2.0/token`; Scope: `https://graph.microsoft.com/Files.ReadWrite.All offline_access`. Enter Client ID + Secret; authorize and confirm Connected status.
  3. **SharePoint Site ID and Drive ID** -- two fixed constants needed in the function. Find them via Graph API (can test via a browser or Postman before wiring into Deluge): `GET https://graph.microsoft.com/v1.0/sites/{hostname}:/sites/{siteName}` for `siteId`; `GET https://graph.microsoft.com/v1.0/sites/{siteId}/drives` for `driveId`. Hardcode both into the function. Confirm `/HR/Onboarding/` root folder exists in SharePoint (or the function will create candidate subfolders directly under wherever you set the root path).
  4. **Discover internal field names** -- the `_downloadUrl` companion field names for all file-upload fields on the Onboarding Staff form are not fully confirmed yet (only `upload_screenshot_of_machine_device_name` is known). Use a temporary debug `return rec.toString()` in `saveFilesToSharePoint` (triggered via a Custom Button against CND219) to dump the full record map and identify all keys ending in `_downloadUrl`. Expected names based on form labels: `proof_of_identity`, `cv`, `2nd_utility_bill`, `police_clearance_certificate`, `screenshot_from_speedtest_net`, `data_protection_act_form`, `referencing_consent_form`, `p45_hmrc_checklist` -- verify and correct each before the real build.
- **Custom Function `saveFilesToSharePoint` -- complete Deluge script (placeholder constants must be filled in after pre-conditions above):**

  ```deluge
  // Replace these two constants after Step 3 above
  siteId = "YOUR_SITE_ID";
  driveId = "YOUR_DRIVE_ID";

  // 1. Look up the Onboarding Staff record
  searchMap = Map();
  searchMap.put("searchField", "CANDIDATE_ID");
  searchMap.put("searchValue", candidateId);
  records = zoho.people.getRecords("Onboarding_Overseas_Staff", 1, 1, searchMap);
  rec = records.get(0);

  // Validate: real record vs. error payload (same gotcha as notifyIT -- API returns error as record 0)
  if (rec.containsKey("code"))
  {
      return "error: getRecords returned " + rec.get("message");
  }

  // 2. Build folder name and path
  firstName = rec.get("CANDIDATE_ID.First_Name");
  lastName  = rec.get("CANDIDATE_ID.Last_Name");
  folderName = candidateId + " - " + firstName + " " + lastName;
  folderPath = "HR/Onboarding/" + folderName;

  // 3. Create candidate folder in SharePoint (conflictBehavior: rename = safe on re-runs)
  folderBody = Map();
  folderBody.put("name", folderName);
  folderBody.put("folder", Map());
  folderBody.put("@microsoft.graph.conflictBehavior", "rename");
  folderResp = invokeurl
  [
      url: "https://graph.microsoft.com/v1.0/sites/" + siteId + "/drives/" + driveId + "/root:/HR/Onboarding:/children"
      type: POST
      parameters: folderBody.toJSONString()
      headers: {"Content-Type": "application/json"}
      connection: "sharepointfileaccess"
  ];

  // 4. File fields: internal Zoho name -> fallback filename if getFileName() returns null
  //    Verify each internal name against rec.toString() output before finalising (see pre-conditions Step 4)
  fileFields = list();
  fileFields.add({"field": "proof_of_identity",                       "fallback": "Proof_of_Identity"});
  fileFields.add({"field": "cv",                                       "fallback": "CV"});
  fileFields.add({"field": "2nd_utility_bill",                         "fallback": "Utility_Bill"});
  fileFields.add({"field": "police_clearance_certificate",             "fallback": "Police_Clearance"});
  fileFields.add({"field": "upload_screenshot_of_machine_device_name", "fallback": "Device_Screenshot"});
  fileFields.add({"field": "screenshot_from_speedtest_net",            "fallback": "Speedtest_Screenshot"});
  fileFields.add({"field": "data_protection_act_form",                 "fallback": "Data_Protection_Form"});
  fileFields.add({"field": "referencing_consent_form",                 "fallback": "Referencing_Consent"});
  fileFields.add({"field": "p45_hmrc_checklist",                       "fallback": "P45_HMRC_Checklist"});

  // 5. Download each file from Zoho People and upload to SharePoint
  for each entry in fileFields
  {
      fieldName    = entry.get("field");
      fallbackName = entry.get("fallback");
      downloadUrl  = rec.get(fieldName + "_downloadUrl");

      if (downloadUrl != null && downloadUrl != "")
      {
          // Download from Zoho People (same pattern as notifyIT_AttachScreenshot)
          fileData = invokeurl
          [
              url: downloadUrl
              type: GET
              connection: "peoplefileaccess"
          ];

          // Prefer the file's own stored name; fall back to the label above
          fileName = fileData.getFileName();
          if (fileName == null || fileName == "")
          {
              fileName = fallbackName;
          }

          // Upload to SharePoint via Graph API simple upload
          uploadResp = invokeurl
          [
              url: "https://graph.microsoft.com/v1.0/sites/" + siteId + "/drives/" + driveId + "/root:/" + folderPath + "/" + fileName + ":/content"
              type: PUT
              parameters: fileData
              connection: "sharepointfileaccess"
          ];
      }
  }

  return "success";
  ```

- **Workflow `Save Candidate Files to SharePoint`** -- Settings > Onboarding > Automation > Workflows, Onboarding Staff form. Trigger: New record is added; Execute: Only once; Criteria: none; Action: Custom Function `saveFilesToSharePoint`, parameter `candidateId` mapped to the Candidate (Onboarding Staff) lookup category (same mapping pattern used for `notifyIT_AttachScreenshot`).
- **Testing sequence:**
  1. Connection smoke-test: temporary `return invokeurl[url:"https://graph.microsoft.com/v1.0/me" type:GET connection:"sharepointfileaccess"];` in the function to confirm the OAuth token flow resolves.
  2. Field name discovery: temporary `return rec.toString();` after `getRecords()`, triggered via "TEST Notify IT (Attach Screenshot)" Custom Button (or a new test button) against CND219 -- read back via Logs > Execution details.
  3. Dry run: add a temporary "TEST Save to SharePoint" Custom Button and trigger against CND219 -- confirm folder `/HR/Onboarding/CND219 - Jay Reck/` created and files appear in SharePoint.
  4. Real workflow test: create a fresh test candidate, submit the Onboarding Staff form, confirm all files land in SharePoint.
- **Known open question:** ITSM subject-line naming convention (ISSUE:zoho 2026-09-07) is still unresolved and unrelated to this feature.
- Status: designed, not yet built -- awaiting pre-conditions (Azure AD app + Zoho Connection + SharePoint IDs + field name confirmation) before creating the Custom Function in Zoho People.

## ASSET:zoho 2026-09-23 -> Zoho People -- saveFilesToSharePoint Custom Function created (stub, pending connection)

- **Custom Function created:** `saveFilesToSharePoint` on the Onboarding Staff form -- Settings > Onboarding > Automation > Actions > Custom Functions. Function ID: `5489000006525005`.
- **Signature:** `String saveFilesToSharePoint(string candidateId)` -- `candidateId` parameter mapped to the Candidate (Onboarding Staff) lookup field (internal: `CANDIDATE_ID`), same mapping as `notifyIT_AttachScreenshot`.
- **Current script:** stub `return "success";` only -- the full script (see `ASSET:zoho 2026-09-21`) cannot be saved yet because Zoho People validates OAuth connection references at Custom Function save time. Since `sharepointfileaccess` does not exist yet, every save attempt with the real script returns `{"failure":"Connection 'sharepointfileaccess' does not exist"}` and the dialog stays open.
- **Script fix during build:** the original planned script used `.toJSONString()` on a Map to build the folder creation POST body. Zoho People's Deluge environment does not support `.toJSONString()` -- the correct approach is to pass the Map directly as `parameters:` in the `invokeurl` block (Deluge serialises Map to JSON automatically when `Content-Type: application/json` is set). Updated script already reflects this fix (see plan above -- `parameters: folderBody` not `parameters: folderBody.toJSONString()`).
- **How the Add Custom Function save works:** the outer Save button sends the full Deluge script (including auto-generated function signature) to `createCustomFunction.zp` as a POST. The server validates referenced OAuth connections by name at save time. The dialog stays open (with no visible error) when the response contains `{"failure":"..."}` -- the failure reason is only visible by intercepting the XHR response, not from the UI.
- **Next step to unlock the real script:** complete the `sharepointfileaccess` pre-conditions (see ASSET:zoho 2026-09-21 -- Step 1 and Step 2), then open this function for editing and save the full script.
- **Remaining pre-conditions before the function is live:**
  1. Register Azure AD app `ZohoPeople-SharePoint` in Azure Portal (Sites.ReadWrite.All permission) -- admin action, outside Zoho.
  2. Create `sharepointfileaccess` Zoho OAuth connection -- Settings > Developer Space > Connections > New Connection > Custom OAuth / Microsoft (clientId + secret + tenantId from Step 1).
  3. Confirm SharePoint Site ID and Drive ID (one-off Graph API lookup) -- hardcode into function constants.
  4. Open `saveFilesToSharePoint` for editing and paste the full Deluge script (see plan above).

## ASSET:zoho 2026-09-23 -> Azure AD app registration -- request sent to 3rd line support (REQ0042511)

- **Request sent 2026-09-23** to 3rd line support / M365 admin (REQ0042511) asking them to register an Azure AD app (`ZohoPeople-SharePoint`) with `Files.ReadWrite.All` delegated permission, redirect URI `https://deluge.zoho.eu/delugeauth/callback`, and share back Tenant ID + Client ID + Client Secret.
- **What was requested of 3rd line:**
  1. Register new App Registration in Azure AD -- name: `ZohoPeople-SharePoint`, single-tenant
  2. Add Redirect URI (Web): `https://deluge.zoho.eu/delugeauth/callback`
  3. API Permissions: Microsoft Graph > Delegated > `Files.ReadWrite.All`, grant admin consent
  4. Create Client Secret (24 months or per org policy), copy value immediately (shown once)
  5. Share back: Tenant ID, Client ID, Client Secret value
- **Unblocks:** `sharepointfileaccess` Zoho connection creation → saving full Deluge script to `saveFilesToSharePoint` (ID: `5489000006525005`) → enabling `Save Candidate Files to SharePoint` workflow.

- **Update 2026-09-30 -- REQ0042511 journal reply from Mohammad (3rd line / M365 admin):**
  - Three questions before proceeding:
    1. **Which SharePoint site?** He can see "HR" and "HR Recruitment" -- wants to scope the app to just one site using `Sites.Selected` rather than `Files.ReadWrite.All`. Asks whether Zoho's connection supports `Sites.Selected` as the OAuth scope.
    2. **Which account for the Zoho-side OAuth connection?** Recommends a dedicated service account so the connection survives staff changes.
    3. **Secret delivery:** will share via Zoho Vault, 12 months with renewal reminder -- no action needed on our side.
  - **Technical notes for reply:**
    - `Sites.Selected` is supported from Zoho's side -- the Scope field in Zoho's custom OAuth connection is free-text; swap `https://graph.microsoft.com/Files.ReadWrite.All offline_access` for `https://graph.microsoft.com/Sites.Selected offline_access`. However, `Sites.Selected` requires an additional Azure step: after app registration Mohammad must also grant the app access to the specific site via Graph API (`POST /sites/{site-id}/permissions`, roles: `write`).
    - Planned folder path is `/HR/Onboarding/...` -- **HR site** is the correct target; confirm before replying.
    - Service account: the account used to authorise the Zoho OAuth connection is the one whose delegated SharePoint permissions back every file write -- needs to be an account that won't be deprovisioned.
    - `hr@transputec.com` is a shared mailbox (multiple HR members including jay.reck@transputec.com access it via delegation, not direct login) -- shared mailboxes cannot authenticate interactively, so cannot be used for OAuth. A dedicated service account is required; deferred to Mohammad to advise or create one.
    - Site ID: no need to provide -- Mohammad has access to both SharePoint sites and can look it up himself.
  - **Status:** pending -- reply drafted 2026-09-30, awaiting send. Key asks: confirm HR site + `/HR/Onboarding/` path; note Sites.Selected extra step (Mohammad grants app access to site after registration); advise on service account since hr@transputec.com is a shared mailbox.

## ASSET:zoho 2026-09-30 -> Zoho People -- end-to-end onboarding flow verified (Welcome Email + Notify IT)

- **Test setup:** temporarily swapped Notify IT email alert recipient from `servicedesk@transputec.com` to `jayreck996@gmail.com` to verify the full flow without impacting the real service desk. CND219 (Jay Reck, ync5389@gmail.com) was deleted (had status "3/3 In progress", "Execute only once" meant it could not be re-triggered anyway).
- **Fresh candidate CND227** created via Track Onboarding > Invite Candidate: Jay Reck, ync5389@gmail.com -- Trigger Onboarding: Yes, Initiate associated workflows: Yes. Status immediately set to "Triggered (yet to accept invite)".
- **Confirmed working end-to-end:**
  - Welcome Email landed at ync5389@gmail.com (portal invite, correct subject/body/links)
  - Notify IT email landed at jayreck996@gmail.com (candidate name + device name, correct subject/body) -- fired automatically without using the TEST button, triggered by the candidate filling in the Machine/Device Name field on the Onboarding Staff form
- **Draft-save vs. submit behaviour clarified:** the "Notify IT" workflow fires on any portal save (including draft save) as soon as the Machine/Device Name is non-empty, not only on final form submission. This is intentional (IT needs to know as early as possible) and the "Execute only once" guard prevents double-firing on subsequent saves. By contrast, `saveFilesToSharePoint` (when built) will trigger on `New record is added` (i.e., form submit), so it captures the final complete record.
- **Recipient restored:** after testing, "Notify IT - Device Details Submitted" email alert recipient changed back to `servicedesk@transputec.com`. Workflow saved successfully ("Workflow updated successfully"). Live production state as of 2026-09-30: Notify IT workflow enabled, recipient servicedesk@transputec.com.
- **Test records remaining:** CND216 (jay.reck@icloud.com, Not Triggered) and CND227 (ync5389@gmail.com, Triggered) are test records in the live candidate list -- can be cleaned up when no longer needed.

## ASSET:zoho 2026-09-30 -> Zoho People -- Notify IT workflow trigger corrected to "Record is created or edited"

- **Root cause of Notify IT not firing for CND228:** CND228 (ync5389@gmail.com) was a fresh candidate whose Onboarding Staff form had never been submitted before -- the very first portal submission is a CREATE event, not an EDIT. The workflow was set to `Existing record is edited` (EDIT only), so that CREATE event was silently skipped and the email never fired.
- **Why earlier tests (CND219, CND227) worked:** those candidates saved a draft first (CREATE, no Machine/Device Name yet → criteria not met) and then submitted again (EDIT, name now filled → criteria met → email fired). CND228 went straight to a full submission with the name already filled, so only CREATE ever happened -- EDIT never came.
- **Admin-side edits do not trigger workflows:** attempted to force-fire by editing the Onboarding Staff record from the admin Operations view (changed Machine/Device Name → saved). Modified Time updated in the record, but no workflow log entry was created. Confirmed: Zoho People onboarding workflows are only triggered by portal-side actions (candidate portal submissions / save-drafts), not admin-side record edits.
- **Fix applied:** changed "Notify IT - Device Name and Screenshot Submitted" workflow trigger from `Existing record is edited` → `Record is created or edited`, sub-option `Execute only once`. This covers both first-time portal submissions (CREATE) and subsequent portal edits (EDIT). Saved -- "Workflow updated successfully" confirmed.
- **Re-test confirmed:** ync5389 did a portal save-draft → three workflow log entries fired at 05:51, 05:55, 05:56, all Successful. Email received at jayreck996@gmail.com. Recipient then restored to `servicedesk@transputec.com` and execution option restored to `Execute only once`.
- **Live production state as of 2026-09-30:** trigger = `Record is created or edited`, execution = `Execute only once`, recipient = `servicedesk@transputec.com`.
- **Cleanup:** CND228 (ync5389@gmail.com) deleted. CND216 (jay.reck@icloud.com) was already absent from the candidate list (no action needed). Candidate list is now empty / clean.

## ASSET:zoho 2026-10-01 -> Zoho People -- Missing candidate records CND146–CND215 (~70 records)

- **Discovery:** Track Onboarding "All" list shows 76 surviving candidates (highest CND145). Test accounts created during previous sessions started at CND216. This leaves a gap of CND146–CND215 (~70 candidate numbers) with no records.
- **Confirmed not caused by our testing:** our test deletions were CND219, CND227, CND228 (our own test accounts). CND216 was already absent before this session. The CND146–215 gap predates all our test activity entirely.
- **CND records survive conversion:** Zoho People does not delete candidate records when a candidate is converted to employee — they persist in Track Onboarding with "Completed" status indefinitely. This confirms the missing CND146–215 records were genuinely deleted outright, not converted.
- **No audit trail:** Zoho People UI exposes no deletion log for candidate records. Workflow & Custom Button Logs only capture workflow executions (oldest entry: 08-09-2026). Reports search for "audit" returns no results. The deletion source, actor, and date are unrecoverable from within the system.
- **Employee records intact:** Employee View export (Employee View (1).csv) shows 349 total records — 159 Active, 105 Resigned, 83 Terminated, 2 Inactive. Active + Inactive = 161, matching Settings page "User License Usage: 159/163". No employees are missing.
- **Candidate-to-employee transition is manual:** conversion requires HR to manually click "Convert to Employee" on the candidate record. "Onboarding Status: Completed" does not auto-convert. Optional auto-trigger for employee onboarding flow exists (Settings → Onboarding → Flow → Preferences) but fires on Date of Joining, not on form completion.
- **Draft email sent to Roann Etan (roann.etan@transputec.com)** 2026-10-01 flagging the gap and requesting confirmation of whether it was intentional cleanup. Also noted absence of audit trail and suggested raising Zoho Support ticket if cause is unknown.
- **Pending candidates still open (not in gap):** CND71 (Andy Vaughan, andyv10@hotmail.com) and CND54 (Kritika Sinha, ksinh2003@yahoo.com) show "Triggered (yet to accept invite)" — both are current active employees who appear not to have completed the candidate portal.

## ASSET:zoho 2026-10-01 -> Zoho People -- Onboarding Staff form submissions: zero records found

- **Onboarding Staff submissions list (Operations → Onboarding → Onboarding Staff, "All Data" filter) shows "No records found"** — confirmed across multiple sessions. The form has columns for Candidate, Employee, Home Address, Reference 1 Email, Proof of Identity/Identification Card, Police Clearance Certificate, Screenshot from speedtest.net, 1st and 2nd Utility Bills.
- **"Completed" Employee Onboarding Status ≠ Onboarding Staff form submitted.** The 33 employees with Onboarding Status "Completed" in the Employee View CSV have completed the *Employee Onboarding flow* — an internal HR data-entry checklist in Zoho People (profile, work details, personal info). This is entirely separate from the Onboarding Staff portal form (form ID 5489000001985025) that collects actual identity documents.
- **Checked inside an employee record (CD30249 Bernadeth Mirto, "Completed"):** Employee record sections are Basic Info, Work, Personal, Summary, Work Experience, Education, Dependent, Related Forms (Company Policy, Exit Details only). No Onboarding Staff form section or document fields anywhere. Her Address field reads "Metro Manila, National Capital Region, Philippines (need full address)" — a placeholder, not a complete submission.
- **Conclusion:** No employee has submitted the Onboarding Staff document form (Police Clearance, Proof of Identity, Utility Bills, Speedtest screenshot, etc.) via the portal. Zero records exist for this form. The "Completed" Employee Onboarding status indicates HR closed out their internal profile checklist, not that documentation was collected.
- **Related gap (Employee View CSV):** 27 active members joined after the Zoho deployment date (earliest triggered/completed join date: 29-03-2021) but were never triggered for Employee Onboarding at all. Combined with 40 triggered-but-incomplete = 67 employees who should have onboarding records but don't.
- **CORRECTION (2026-10-05):** the "No records found" finding was incorrect — see ASSET:zoho 2026-10-05 -> Zoho People -- Onboarding Staff form submissions found in completed candidate records. The zero-records result was likely caused by a filter or view access issue when navigating via Operations → Onboarding → Onboarding Staff directly. Completed candidate records accessed via Operations → Onboarding → Track Onboarding → All do show linked Onboarding Staff form submissions with actual files.

## ASSET:zoho 2026-10-01 -> Zoho People -- saveFilesToSharePoint ruled out as cause of CND gap

- **Question investigated:** whether the `saveFilesToSharePoint` custom function (Onboarding Staff form, Settings → Onboarding → Automation → Actions → Custom Functions) could have deleted the CND146–215 candidate records as a side-effect of archiving to SharePoint.
- **Finding: the function has never run real code.** Version history shows a single Version 1.0 created 2026-09-23 04:53 GMT, always containing only `return "success";`. The real Deluge script (documented in ASSET:zoho 2026-09-21) could not be saved because Zoho People validates OAuth connection references at save time — `sharepointfileaccess` does not exist yet, causing `{"failure":"Connection 'sharepointfileaccess' does not exist"}` on every save attempt.
- **Pre-conditions still incomplete as of 2026-10-01:**
  - Azure AD App Registration: pending REQ0042511 (Mohammad, 3rd line). Mohammad replied 2026-09-30 with three clarifying questions; reply drafted but not yet sent.
  - `sharepointfileaccess` Zoho OAuth Connection: not created.
  - `Save Candidate Files to SharePoint` Workflow: not yet created (designed in ASSET:zoho 2026-09-21, blocked by above).
- **Full automation stack reviewed (Onboarding module):** Workflows — 2 only (email-only, no delete actions). Webhooks — none configured. Custom Functions — `notifyIT_AttachScreenshot` (read + sendmail only, no deletion) and `saveFilesToSharePoint` (stub). No delete action exists anywhere in the Onboarding automation layer.
- **Conclusion:** the SharePoint automation cannot have caused the CND146–215 gap. It has never executed against any candidate record. The gap cause remains unrecoverable from within Zoho People (no audit trail).

## ASSET:zoho 2026-10-01 -> Zoho People -- Zoho Support ticket raised for missing CND146–215 records

- **Action:** Email sent to `support@eu.zohocorp.com` from jay.rock@transputec.com (2026-10-01) requesting data recovery for ~70 missing candidate records (CND146–CND215) in Track Onboarding.
- **Key points raised in ticket:** records were confirmed present earlier this week (around 27–29 Sep 2026); sequential gap from ~CND138 to CND216+; no automation or custom function in account performed delete actions; Workflow & Custom Button Logs show no relevant activity; Zoho People exposes no audit trail for manual UI deletions.
- **Requests made:** (1) check whether records still exist and can be restored/recovered; (2) advise if Zoho backend audit logs show what action caused the deletion and when.
- **Zoho EU status page checked (2026-10-01):** no incident affecting EU Zoho People in the Sep 27–Oct 1 window. Sep 27 had planned EU DC maintenance but Zoho People was not among affected components. Sep 29 had a 30-min outage (EU Zoho Analytics + others) — outages do not delete records. Oct 1, Sep 30, Sep 28: no incidents.
- **Update 2026-10-02:** Tanzeel (Senior Product Support Engineer) replied suggesting Activity Log check — led to confirming the deletion details (see ASSET:zoho 2026-10-02 -> Activity Log confirms CD30372 deleted 99 records).
- **Update 2026-10-06:** Tanzeel confirmed the recovery request has been raised with their internal team. Separate email same day confirmed deletion details: records were manually deleted on 30-09-2026 at 05:08:24 CEST from account jay.reck@transputec.com. Zoho confirmed ability to restore the Candidate records but stated associated onboarding data cannot be recovered.
- **Update 2026-10-07 — RESOLVED:** Zoho restored the 99 Candidate records (CND146–CND215). **However, all onboarding-related data associated with those records has been permanently deleted and cannot be recovered.** This includes all Onboarding Staff form submissions and attached files (Proof of Identity, Utility Bills, Speedtest screenshots, Personnel Questionnaires, Data Protection Act Forms, etc.) for any of those candidates who had completed the onboarding flow before deletion. To resume onboarding for any of the restored candidates, the onboarding process must be retriggered from scratch. **Status: CLOSED — records restored, onboarding data permanently lost.**

## ASSET:zoho 2026-10-02 -> Zoho People -- Activity Log confirms CD30372 (Jay Reck) deleted 99 Candidate records on 30 September

- **Finding:** Operations > Data Administration > Activity Log, filtered by Entity: Records / Action: Delete / Form name: Candidate, shows three deletion events on 30 September by CD30372 - Jay Reck:
  - 04:09 -- **99 records** deleted (the CND146–215 gap)
  - 04:56 -- 1 record deleted (likely CND228, the intentional test-record cleanup)
  - 06:10 -- 1 record deleted (likely CND219 or CND227, the intentional test-record cleanup)
- **Cause identified:** the 99-record bulk deletion at 04:09 on 30 September accounts for the entire CND146–215 gap. Actor confirmed as Jay Reck (CD30372). The deletion is not documented as intentional in the 2026-09-30 ASSET log (only CND219, CND227, CND228 are mentioned as deliberate cleanups) -- likely an accidental bulk select-all + delete while navigating the Track Onboarding list during testing.
- **Surfaced via:** Zoho Support ticket response from Tanzeel (Senior Product Support Engineer) dated 2026-10-02, who suggested checking the Activity Log -- previously missed because the standard UI search for "audit" returned no results and the Workflow & Custom Button Logs only cover workflow executions.
- **Recovery path:** Zoho Support ticket (ASSET:zoho 2026-10-01 -- Zoho Support ticket raised) now has the exact deletion details to request a backend restore: date 30 Sep 2026, time ~04:09, actor CD30372, form Candidate, 99 records. Reply to Tanzeel with these findings pending.

## ASSET:zoho 2026-10-02 -> Zoho People -- Recycle Bin: deleted candidate records may be recoverable

- **Discovery:** Zoho People has a native Recycle Bin (Operations > Data Administration > Recycle Bin). Custom form records -- including Candidate/Onboarding records -- are retained there for 30 days after deletion by default.
- **Relevance:** the missing CND146–CND215 records (see ASSET:zoho 2026-10-01 -> Zoho People -- Missing candidate records CND146–CND215) were identified 2026-10-01; if deleted within the previous 30 days, they may still be recoverable from the bin before the window closes.
- **How to restore:** Operations > Data Administration > Recycle Bin > select the Candidate/Onboarding form > check the desired records > click Restore (confirm twice). Records appear in the bin approximately 30 seconds after deletion.
- **Retention:** 30 days default; configurable via Settings > Manage Accounts > Organization Setup > Organization Policy > Recycle Bin Preference.
- **Limitation:** records from core system forms (leave, attendance, performance) cannot be restored -- Candidate/Onboarding custom form records are restorable.
- **Next action:** check the Recycle Bin immediately. If CND146–215 are present, restore them and close the Zoho Support ticket (ASSET:zoho 2026-10-01 -- Zoho Support ticket raised).

## ASSET:zoho 2026-10-05 -> Zoho People -- Candidate Onboarding form field inventory (complete, corrected)

- **Purpose:** full audit of all fields across the Candidate Onboarding flow. Previous version of this entry was incomplete and contained an incorrect file loss assessment (corrected below).
- **Source:** Settings > Onboarding > Candidate Onboarding > Flow > Preview Onboarding Flow + live record inspection of CND132 (Khushboo Masih, Completed) via Operations → Onboarding → Track Onboarding → All.

**Your Details (Profile) — stored directly on the CND candidate record:**
- First Name, Last Name, Mobile, Email ID, Company Email (text fields)
- **Photo** — file upload (JPG, PNG, GIF, JPEG; max 5 MB)
- Street Address, City, State/Province, Country (address fields)
- Source of hire, Department, Tentative Joining Date, Location, Title (professional fields)
- **Offer Letter** — file upload (any format; max 5 MB)

**Onboarding Staff form — separate linked form submission (accessed via candidate record → "Onboarding Staff" link):**
- Candidate (lookup to CND record), Home Address, Personal Email Address, Job Role Offered, Reference 1 Email, Reference 2 Email (text/textarea fields)
- Next of Kin Name & Contact Number (text field)
- Machine/Device Name — If applicable write Company Issued Laptop (text field)
- **Proof of Identity / Identification Card** — file upload (max 5 MB, required)
- **1st Utility Bill (address included)** — file upload (max 5 MB, required)
- **2nd Utility Bill (address included)** — file upload (max 5 MB, required)
- **P45/HMRC Checklist (UK Staff Only)** — file upload
- **CV** — file upload
- **Police Clearance Certificate (PCC — Remote Workers Only)** — file upload (max 5 MB)
- **Screenshot from https://www.speedtest.net** — file upload (max 5 MB, required)
- **Personnel Questionnaire** — file upload (max 5 MB, required)
- **Data Protection Act Form** — file upload
- **Referencing Consent Form** — file upload
- **Upload Screenshot of Machine/Device Name** — file upload

**How the two records relate:** from Track Onboarding the candidate entry is the parent; the Onboarding Staff form record is a child linked via the Candidate lookup. Both are visible together under the candidate's "Profile and Other Forms" view. From the user's perspective the candidate record contains all documents.

**File loss assessment for the 99 deleted CND146–215 records (corrected 2026-10-05):**
- Photo and Offer Letter on the CND profile: lost if those candidates had completed their profile before deletion.
- Onboarding Staff form documents (ID, Utility Bills, Speedtest, Personnel Questionnaire, Data Protection Act Form, etc.): **potentially lost** — Onboarding Staff form submissions DO exist for completed candidates (see ASSET:zoho 2026-10-05 -> Zoho People -- Onboarding Staff submissions found). Whether those linked records were cascade-deleted when the 99 CND records were deleted is unknown. If cascade-deleted, identity documents and compliance files for any of those 99 candidates who had submitted the form are permanently gone.
- **Recommendation:** the Zoho Support ticket reply to Tanzeel should explicitly ask (1) whether a backend restore recovers the CND records and all linked Onboarding Staff form records including file attachments, and (2) whether the Onboarding Staff records for CND146–215 still exist as orphaned records if not cascade-deleted.

## ASSET:zoho 2026-10-05 -> Zoho People -- Onboarding Staff submissions found in completed candidate records

- **Finding:** Onboarding Staff form submissions DO exist. CND132 (Khushboo Masih, khushboomasih193@gmail.com) has a completed submission with the following files attached: Passport (1).pdf (Proof of Identity), 1.odt (1st Utility Bill), 2.odt (2nd Utility Bill), speedtest.png (Speedtest screenshot), PERSONNEL QUESTIONNAIRE (1)(1).pdf, DATA PROTECTION ACT FORM (1).doc.
- **How found:** Operations → Onboarding → Track Onboarding → All → clicked Khushboo Masih's entry → "Profile and Other Forms" panel → clicked "Onboarding Staff" link. Form record ID: 5489000003676049, added 29-07-2024, modified by EMP798 Roann Etan 29-07-2024.
- **Correction to:** ASSET:zoho 2026-10-01 -> Zoho People -- Onboarding Staff form submissions: zero records found. That entry was wrong — the zero-records result was likely due to a filtered view when navigating directly via Operations → Onboarding → Onboarding Staff. The form has active submissions accessible via the candidate record route.
- **Implication for the 99 deleted records:** any of the 99 candidates (CND146–215) who had reached "Completed" status and submitted the Onboarding Staff form would have had identity documents, utility bills, and compliance files attached. Whether those Onboarding Staff form records were cascade-deleted with the CND records is the critical unknown — see corrected loss assessment in ASSET:zoho 2026-10-05 -> Candidate Onboarding form field inventory.

## ASSET:zoho 2026-10-07 -> Zoho People -- Onboarding Status Report: restored records analysis (Onboarding Status Report (2).csv)

- **Source:** Onboarding Status Report (2).csv export, 25 records.
- **Context:** reviewed to assess which of the 99 deleted CND146–215 records were restored by Zoho and what their onboarding status was at time of deletion.

**Restored records from the deleted CND146–215 range (11 of 99):**

| Candidate ID | Name | Email | Status |
|---|---|---|---|
| CND146 | Roble Farah | roblefarah@hotmail.co.uk | Triggered |
| CND147 | Yusra Albeiti | yalbeitii@gmail.com | Triggered |
| CND148 | Ryuma Rocco | ryumarocco@hotmail.com | Triggered |
| CND169 | Chanay Blomkamp | chanayjacobs@ymail.com | Triggered |
| CND174 | Andy Joseph | josephandy581@gmail.com | Triggered |
| CND188 | Giovani Bongiolo | giobongiolo@gmail.com | Triggered |
| CND189 | Nico Goosen | dr.nicogoosen@outlook.com | Triggered |
| CND199 | Zadel De Wit | zadeldewitt93@gmail.com | Triggered |
| CND209 | Mohammed Radman | mohammedradman11@gmail.com | **Completed by candidate** |
| CND214 | Alfredo Jr Salipot | ajr.as@outlook.com | **Completed by candidate** |
| CND215 | Mahmoud Aly | mahmoudaly82016@outlook.com | **Completed by candidate** |

**The remaining 88 of the 99 deleted records do not appear** — likely stub records never accepted by the candidate (no profile data to restore) or absent from this export's filter.

**Critical — 3 candidates with "Completed by candidate" status (CND209, CND214, CND215):** these had fully completed the onboarding flow, meaning they submitted the Onboarding Staff form with all attached documents (Proof of Identity, Utility Bills, Speedtest screenshot, Personnel Questionnaire, Data Protection Act Form). Their profile records are restored but all submitted documents are permanently lost. These 3 candidates must be contacted to resubmit their full document set.

**8 candidates with "Triggered" status:** accepted their invite and started the flow but had not submitted the Onboarding Staff form. Onboarding must be retriggered; no documents were lost for these candidates.

**Other records in the CSV (not from the deleted range):** CND101, CND103, CND116 (pre-deletion survivors), CND218–CND230 (post-deletion new/test records), CND111, CND71, CND54 (pre-existing active candidates).

## ASSET:zoho 2026-10-07 -> Zoho People -- data recovery policy research: no client-facing policy for Candidate record deletion

- **Purpose:** researched whether Zoho People has a published data recovery or retention policy for Candidate records and linked data, to support a formal policy request to Zoho.
- **Sources checked:** Zoho People Help (help.zoho.com), Zoho Community forums.

**Findings:**
- **Official documentation (Candidate Onboarding Records help article)** states only: *"All data related to the selected candidates will be deleted from Zoho People."* No mention of recovery options, recycle bin support, retention window, or warning about linked records and attached files being destroyed.
- **Recycle Bin coverage gap (confirmed in our own system):** Zoho People Recycle Bin (Operations → Data Administration → Recycle Bin) lists Employee, Exit Details, and Onboarding Staff forms — but NOT Candidate records. Candidate is a system-level module, not a custom form, so it bypasses the standard 30-day bin entirely.
- **Employee records DO have recycle bin support** (30-day retention, restorable from UI). Candidate records do not — recovery requires a backend support ticket to Zoho, with no self-service path.
- **Community precedent:** a separate Zoho user reported accidentally deleting 18 candidates and attempted recovery (help.zoho.com community) — no published resolution, suggesting this is a recurring known pain point with no consistent self-service fix.
- **No published policy exists** for: (1) retention of deleted Candidate records, (2) recovery of linked form submissions when a parent Candidate record is deleted, or (3) preservation of file attachments on deleted records.

**Conclusion:** Zoho People has an undocumented gap — Candidate records and their linked Onboarding Staff data (identity documents, compliance files) are permanently deleted with no UI recovery path and no client-facing policy. Zoho's backend team can restore records on a case-by-case basis (as confirmed in our incident) but this is not documented and not guaranteed. A formal request for Recycle Bin coverage of Candidate records and linked data is justified and supported by the documentation gap.

## ASSET:zoho 2026-10-07 -> Zoho People -- HR process recommendation: convert candidates to employees earlier to reduce data-loss risk

- **Observation:** the root structural risk in the 99-record deletion incident is that people remained as Candidate (CND) records throughout the entire onboarding document-collection phase. Candidate records are NOT covered by the Recycle Bin and have no self-service recovery path. Employee records ARE in the Recycle Bin (30-day retention, restorable from UI).
- **Current process:** Candidate record is created at invite → candidate submits Onboarding Staff documents → HR reviews → HR eventually clicks "Convert to Employee" manually (no auto-trigger on form completion). The conversion is entirely manual and timing is undefined — candidates appear to remain in CND status for weeks or months after submitting all their documents.
- **Risk window:** any period where a candidate has submitted identity documents and compliance files but is still a CND record is a period where accidental deletion causes permanent unrecoverable data loss (as confirmed by this incident).
- **Recommendation:** convert to Employee status at the point of onboarding form submission (or offer acceptance), not at day-1 start. Zoho People supports pre-start employees — an Employee record can exist before the Date of Joining with a status that reflects "not yet started." This moves the record into the protected tier (Recycle Bin coverage) as soon as the compliance-critical documents are on file.
- **How to apply:** raise with HR to define a policy for when conversion happens (e.g., "convert when Onboarding Staff form is submitted and Proof of Identity is received"). Could be supported by a Workflow that flags/notifies HR to convert when the Onboarding Staff record is created. Auto-conversion via Zoho Onboarding Flow Preferences exists but fires on Date of Joining, not on form completion — a manual or workflow-triggered conversion earlier in the cycle would close this gap.
- **Why this was not raised before:** the Recycle Bin gap for Candidate records only became apparent after the 30 Sep 2026 deletion incident and the subsequent research into Zoho's recovery policy.
