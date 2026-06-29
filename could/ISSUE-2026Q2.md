# ISSUE Log - Zoho

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
