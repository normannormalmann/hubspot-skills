---
name: hubspot-sequences
description: Build, edit, test and auto-enroll HubSpot sequences (Sales Hub), including email templates, call tasks, delays, "pause until task is completed", reply-in-thread and GDPR subscriptions. Always use this when the user talks about HubSpot sequences, cadences, call/email follow-up flows for setters or sales reps, enrolling contacts into sequences via n8n or the API, sequence templates, or errors like INVALID_SEQUENCE_STRUCTURE, TEMPLATES_NOT_FOUND or UNSUBSCRIBED – even if they only say "cadence", "follow-up sequence" or "automatic call tasks".
---

# Building and auto-filling HubSpot sequences

Hands-on knowledge from building real sales cadences. The hard rules below come from actual failures; ignoring them costs hours.

## Step 0: Clarify the configuration

Before creating anything, read `references/configuration.md`.
- If it is already filled in, use those values.
- If it still contains placeholders (`<…>`), determine the values as described there (via the API or by asking the user) and offer to fill in the file. Never guess IDs.
- The legal basis for the sales subscription (rule 6) is **always decided by the user or their company** – never choose one yourself, ask.

## The 7 hard rules

1. **The public API can only read sequences and enroll contacts, not create them.** Creating/editing only works in the UI or via the internal endpoints from a logged-in browser tab (e.g. Claude in Chrome, `fetch` with CSRF header). See `references/internal-api.md`.
2. **For automatic enrollment (API/n8n), step 0 must be an email.** If the sequence starts with a task, `POST /automation/v4/sequences/enrollments` fails with `INVALID_SEQUENCE_STRUCTURE`. Fix: first email = acknowledgement ("Thanks for your inquiry, I'll call you shortly"), then call 1 right after with delay 0.
3. **An email must not follow a task with delay 0** (same error). Use at least 1 business day (`delayMillis: 86400000`) between a task and the following email.
4. **Reply in thread (`threadEmailToStepOrder`) only as a chain:** each base email may have exactly one reply ("Multiple emails threading on the same base step"). Build a chain: E1 ← E2 ← E3, not E1 ← E2 and E1 ← E3.
5. **Email templates need the content permission, not just `private:false`.** Otherwise you get `TEMPLATES_NOT_FOUND` as soon as another user (e.g. the setter) enrolls as sender. Share via `PUT /api/contentpermissions/v1/permissions` with `visibleToAll: true` (see reference).
6. **GDPR-enabled portals: contacts need the sales/one-to-one subscription**, otherwise `Contact has unsubscribed`. HubSpot also reports `NOT_SUBSCRIBED` for subscriptions that were never set – detect real opt-outs via the contact properties `hs_email_optout` and `hs_email_optout_<subscriptionId>`, not via the status. Set the legal basis (`legalBasis`) as decided by the company (see `references/configuration.md`).
7. **The sender needs a connected inbox.** `senderEmail` must be the inbox connected in HubSpot for `userId`; users without one → `PUBLIC_ENROLL_NO_CONNECTED_EMAILS`.

More facts:
- "Pause sequence until task is completed" = dependency `TASK_COMPLETION` (the following step `requiredByStepOrder` depends on the task's `reliesOnStepOrder`).
- Sequences have no on/off switch; they only affect enrolled contacts.
- A contact can only be in one sequence at a time (`CONTACT_ALREADY_ENROLLED`).
- Sequences and templates belong to the user who created them; ownership cannot be changed via API. Sharing with everyone is sufficient.
- Token format in templates: `{{ contact.firstname }}`, `{{ contact.lastname }}`, custom properties as `{{ contact.<internal_property_name> }}`. Avoid dropdown properties (e.g. salutation) as tokens – the internal value may show up.
- The internal endpoints are undocumented and may change without notice. On unexpected errors, first check whether the UI itself still uses the same call (network tab).

## Workflow

1. **Define the cadence** (table: step, business day, type, content). Apply rules 2–4 right away.
2. **Create email templates** (`POST /api/sales-templates/v3/templates` with `{template:{name, subject, body}}`), then set the **content permission** (rule 5).
3. **Create the sequence**: most reliable is to create a first version in the UI (Automation → Sequences → Start from scratch → one step → Save → share with "Everyone"), then write all steps via `PUT /api/sequences/v3/sequences/update-with-permissions/<id>` with a complete `sequenceEgg`. Further sequences via `POST /api/sequences/v3/sequences/create` with `{sequenceEgg}` (copy without IDs).
4. **Verify**: `GET /api/sequences/v2/sequences/<id>` and print the steps in short form (E0 C0 C1* …).
5. **Test enrollment** with your own test addresses (plus addresses like `name+t1@domain`), set the subscription first. On structure errors, narrow down with mini sequences (email only / email+call / call+email …).
6. **n8n enrollment** in test mode, then live (pattern in `references/n8n-enrollment.md`).
7. **Exit rules**: replies and booked meetings end the sequence automatically; other exit reasons (e.g. lead status disqualified, new deal) via a HubSpot workflow "Unenroll from sequence".
8. **Clean up**: test sequences, test contacts, test tasks assigned to the sender.
9. **Update the configuration**: record new sequence, template and workflow IDs in `references/configuration.md`.

## References

- `references/configuration.md` – your own portal, user, subscription, sequence and template IDs (template to fill in)
- `references/internal-api.md` – endpoints, payload formats, JS snippets for the browser tab
- `references/n8n-enrollment.md` – n8n enrollment flow incl. subscription/legal basis
- `references/example-cadence.md` – complete example of a rule-compliant cadence with builder plan
