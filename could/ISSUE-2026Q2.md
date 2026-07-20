# ISSUE Log - Zoho

## ISSUE:zoho 2026-07-20 -> Zoho Mail -- no Zoho Mail connection available for Flow (Invalid Email ID / null AccountId)

**Symptom:** Creating a new Zoho Mail connection in Zoho Flow (for a Send Email action) fails with "Invalid Email ID, null AccountId cannot be obtained for the given Email ID".

**Cause:** Zoho Mail is not provisioned as a mailbox for the authorizing account/org (confirmed: mail.zoho.eu/createMailAccount returns page not found for jay.reck@transputec.com) -- org likely uses a different email service, so the Zoho Mail app has no account to attach to.

**Resolution:** Skip the separate Zoho Mail action. Use the "Mail content" field available directly on other Zoho actions that already send email natively (e.g. Zoho Learn's "Invite user to hub" action) to include any additional links/text, instead of adding a standalone Send Email step.

## ISSUE:zoho 2026-07-16 -> Zoho Learn -- new hire onboarding docs and forms sent manually via email

**Limitation:** HR admin currently sends essential onboarding documents to new hires via email, and new hires must email completed forms back. No automated delivery or centralized submission point exists.

**Impact:** Manual email round-trips slow down onboarding and leave no single source of truth for submitted new-hire forms/documents -- everything lives in inboxes.

**Resolution:** See ASSET:zoho 2026-07-16 -> Zoho Flow -- automated onboarding doc delivery and form submission for the streamlined replacement (Flow + WorkDrive + Forms, no email required).

## ISSUE:zoho 2026-06-30 -> Zoho Learn -- user sees Nothing in your bucket yet (not in hub)

**Symptom:** User logs in but sees no courses.
**Cause:** User not added to the Transputec hub. Common for new starters.
**Resolution:** Admin > hub settings > invite user by Transputec email > user clicks accept > logs in with Transputec credentials.
**Standard response template:** HR Systems Administrator Guide, Section 2.2.

## ISSUE:zoho 2026-06-30 -> Zoho Learn -- course stuck below 100% completion (known glitch)

**Symptom:** User completed all lessons but progress bar stays below 100%. Some lessons show next lesson only, no complete option. Course shows complete in admin view but not for the user.
**Cause:** Known Zoho Learn glitch -- triggered when new lessons are added to a course after users have already started it.
**Resolution:** Admin > affected course > user progress report > identify stuck lesson (shows In Progress) > reset that lesson > user accesses and marks complete.
**Standard response template:** HR Systems Administrator Guide, Section 2.3.

## ISSUE:zoho 2026-06-23 -> Zoho Learn -- native email notifications do not support file attachments

**Limitation:** Zoho Learn built-in email notification system (Admin > Email Notifications) supports template customisation and placeholders but does not support preset file attachments natively.

**Workaround:** Host files in Zoho WorkDrive and embed the download link in the email template body. Functionally equivalent to an attachment without requiring external tools.

**If true attachment required:** Use Zoho Flow (no-code drag-and-drop) to trigger an email with an attached file on a Zoho Learn event (enrollment, completion, etc.).

## ISSUE:zoho 2026-06-23 -> Zoho Learn -- no programming language required for email and LMS setup

**Confirmed:** Email templates, placeholder variables, automation triggers, and Zoho Flow workflows are all configured via UI only. No code required.

**Exception:** Zoho Learn API exists for programmatic bulk actions (enrollments, completion data) but is optional and not needed for standard email/attachment workflows.
