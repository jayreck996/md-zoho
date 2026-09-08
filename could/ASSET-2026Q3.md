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
