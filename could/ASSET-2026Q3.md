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
- Status: candidate record exists, onboarding not yet triggered, Welcome Email not yet sent

## ASSET:zoho 2026-08-27 -> Tooling -- claude-in-chrome browser extension/MCP (used for this Zoho work)

- Not a Zoho product -- documented here because this is the tool actually used to configure the Zoho People onboarding email template in this quarter's entries above, for anyone reproducing or continuing this work
- What it is: an Anthropic-provided Chrome extension that connects the browser to a Claude session via MCP tools, letting Claude view and drive a real Chrome tab -- screenshots, clicks, typing, reading the page's DOM/accessibility tree, running JS -- instead of an invisible/headless browser
- Why it was needed here: this environment's default browser-automation path (a separate "wmux browser" panel) wasn't rendering Zoho People's page content correctly -- it kept returning the outer app shell instead of the actual page. Switched to claude-in-chrome and it worked reliably for the rest of the People configuration work.
- Session/permission model: Claude opens and controls tabs inside a dedicated tab group scoped to that session -- not arbitrary access to all open browser tabs/windows. Claude does not see or handle login credentials -- the user logs in manually in that tab; Claude only interacts with the page afterward.
- Tools available (MCP), as used this session: `tabs_context_mcp` (list/create the session's tab group), `navigate`, `computer` (mouse/keyboard/screenshot), `read_page` / `find` (accessibility tree), `javascript_tool` (run JS in page context -- used repeatedly here to verify what was actually saved server-side vs. what the on-screen preview showed), `file_upload` (attach local files to a file input, bypassing the native OS file picker), `resize_window`, plus `read_console_messages` / `read_network_requests` / `gif_creator` (not used this session)
- Install/setup: install the Claude Chrome extension from the Chrome Web Store, sign in, then grant it permission for the sites Claude should access. Exact install flow and availability can change over time -- check Anthropic's current documentation rather than relying solely on this note when setting it up fresh.
