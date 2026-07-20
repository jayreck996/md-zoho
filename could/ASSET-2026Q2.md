# ASSET Log - Zoho

## ASSET:zoho 2026-07-20 -> Zoho Flow -- "New Hire Setup" flow built and tested (Form trigger + invite with embedded links)

- Built as: Trigger = Zoho Forms "Form entry submitted" on a "New Hire Setup" form (HR fills in new hire's First name/Last name/Email) -- used instead of a bare manual trigger, since Forms gives structured captured data and a record of submissions
- Action = Zoho Learn "Invite user to hub" (Transputec Limited hub), mapped from trigger.Email/Name_First/Name_Last
- Onboarding form link + WorkDrive docs link embedded directly in the invite action's "Mail content" field, rather than a separate Send Email step -- avoids needing a separate Zoho Mail app connection (see ISSUE:zoho 2026-07-20 -> Zoho Mail -- no Zoho Mail connection available for Flow)
- Result: one HR form submission -> one automated email to new hire containing hub invite plus onboarding links, no separate email leg needed
- Tested via Test & debug using own real email as the new hire -- confirmed real invite and custom mail content email were received
- Status: built and tested working

## ASSET:zoho 2026-07-16 -> Zoho Flow -- Test & debug mode does not prevent real side effects (correction)

- Correction to ASSET:zoho 2026-07-16 -> Zoho Flow -- no unified sandbox, use Test & debug mode instead: that entry stated Test & debug "does not affect live data or send real emails" -- confirmed false by a live test run
- Test run evidence: triggering the flow manually via Test & debug still executed the real Zoho Learn "Invite user to hub" action for real (`"result":"success"`, `"successUserCount":"1"`, real entry in `invitedUsers`) -- a genuine hub invite was sent, not a simulation
- Test & debug only changes how the flow starts (manual run with sample/recent data instead of automatic on real form submissions) -- every action inside the flow still executes with real side effects every time, no dry-run exists
- Safe testing practice: always use your own real email/data as the test "new hire" so any real side effect (e.g. an actual hub invite) only ever touches yourself, never a real employee
- Secondary gotcha caught in the same test: a typo'd email domain (e.g. `trancputec.com` instead of `transputec.com`) still returned `"result":"success"` -- the API does not validate that the domain exists, so a successful result does not confirm actual delivery. Double-check submitted test data, don't just trust a success response

## ASSET:zoho 2026-07-16 -> Zoho Flow -- no unified sandbox, use Test & debug mode instead

- No dedicated staging/sandbox environment exists across Learn, Forms, and Flow (unlike Zoho CRM, which has a separate Sandbox feature on Enterprise+ plans, or Zoho Sign's sandbox added April 2026)
- Zoho Flow has a built-in Test & debug mode: run the flow against sample or recent test data, inspect each step's input/output, and re-run before activating -- does not affect live data or send real emails
- Zoho Forms: keep the form unpublished/inactive while testing instead of relying on a separate environment
- Applies to the onboarding automation in ASSET:zoho 2026-07-16 -> Zoho Flow -- corrected trigger design for onboarding automation -- validate via Test & debug before flipping the flow live

## ASSET:zoho 2026-07-16 -> Zoho Flow -- corrected trigger design for onboarding automation

- Correction to ASSET:zoho 2026-07-16 -> Zoho Flow -- automated onboarding doc delivery and form submission: Zoho Flow's Zoho Learn integration only exposes actions (invite user, add member, create course), not a trigger for new-learner-enrolled -- Flow cannot watch Zoho Learn and auto-fire on enrollment
- Trigger instead: HR admin manually starts the flow (short "New Hire Setup" trigger with name/email) -- admin-initiated, not event-detected
- Flow actions: (1) invite new hire to Zoho Learn hub via Flow's Zoho Learn action, (2) send new hire a single Zoho Form link
- Doc delivery folded into the form itself: WorkDrive doc link(s) added as a text/instructions field inside the Zoho Form (no native "download field" in Zoho Forms, but a text field with a link works) -- new hire reads/downloads docs and submits the same form, no separate email leg
- Result: one admin action, one link sent to new hire, no attachments, no separate WorkDrive email step
- Status: proposed -- not yet built in Zoho Flow

## ASSET:zoho 2026-07-16 -> Zoho Flow -- automated onboarding doc delivery and form submission

- Trigger: new hire enrolled in Transputec hub (Zoho Learn event)
- Action 1: Zoho Flow auto-sends essential onboarding docs to new hire via WorkDrive share link (native email attachments not supported in Zoho Learn)
- Action 2: New hire submits required info via Zoho Form instead of replying by email -- form routes directly to HR's dashboard/view
- Result: HR admin's only manual step is initial enrollment; document delivery and submission collection are automated, no email round-trips
- Status: proposed -- not yet built in Zoho Flow

## ASSET:zoho 2026-06-30 -> Zoho Learn -- platform overview and access

- **Login:** https://www.zoho.com/learn/login.html
- Transputec LMS -- all staff enrolled via Transputec hub
- Admin responsibilities: course assignments, user issues, content updates

## ASSET:zoho 2026-06-30 -> Zoho Learn -- assigning courses to new starters

- Invite new starter to Transputec hub via their Transputec email (Admin > hub settings)
- Once invitation accepted, assign mandatory onboarding courses to their account
- Confirm with HR Manager which courses are required for their role

## ASSET:zoho 2026-06-30 -> Zoho Learn -- updating course content

- Retrieve latest video from Vimeo or SharePoint
- Zoho Learn > relevant course > lesson > replace video link > save > confirm video plays
- Always confirm with HR Manager before making changes to existing content

## ASSET:zoho 2026-06-30 -> Zoho Forms -- platform overview and access

- **Login:** https://www.zoho.com/forms/login.html
- Hosts all HR forms used within Zoho Learn -- Reference requests, Remote working checklists, KPI Review Form
- Admin creates, updates, and maintains forms as directed by HR Manager

## ASSET:zoho 2026-06-30 -> Zoho Forms -- creating a new form

- Zoho Forms > New Form > select template or start from blank
- Add required fields and configure routing to correct HR team member
- Test the form before sharing with staff

## ASSET:zoho 2026-06-30 -> Zoho Forms -- updating an existing form

- Zoho Forms > locate form > make changes > save and publish
- Notify HR Manager once updated

## ASSET:zoho 2026-06-23 -> Zoho Learn -- email template setup

- **Location:** Admin > Email Notifications inside Zoho Learn
- **Triggers available:** Enrollment, course completion, due date reminders
- **Supported placeholders:** `{{learner_name}}`, `{{course_name}}`, `{{portal_link}}`
- **Configuration:** Point-and-click UI, no code required

## ASSET:zoho 2026-06-23 -> Zoho Learn -- attachment workaround via WorkDrive link

- **Method:** Host file in Zoho WorkDrive -> copy share link -> paste into email template body
- **Result:** Learner clicks link to download -- functionally same as attachment
- **No external tools needed:** Fully onsite within Zoho ecosystem
- **No code required**

## ASSET:zoho 2026-06-23 -> Zoho Learn -- true attachment flow via Zoho Flow

- **Tool:** Zoho Flow (drag-and-drop automation, no code)
- **Trigger:** Zoho Learn event (e.g. course completed, learner enrolled)
- **Action:** Send Zoho Mail with file attached
- **Use when:** Actual file attachment is required rather than a link
- **No programming language required**
