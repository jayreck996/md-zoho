# ASSET Log - Zoho

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

