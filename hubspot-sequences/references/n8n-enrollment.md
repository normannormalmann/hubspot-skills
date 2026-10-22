# Automatic enrollment via n8n

Flow (proven in practice):

1. **Configuration** (Code node): `mode` test|live, `testEmails`, `windowHours`, mapping source → sequence ID, setter owner ID, setter user ID, sender inbox, subscription ID.
2. **Find candidates**: `POST /crm/v3/objects/contacts/search`; filters owner = setter, lead status NEW, source in mapping, email present, `num_associated_deals` empty; in test mode use `email IN testEmails` instead of the time window. Properties incl. `hs_sequences_is_enrolled`, `hs_latest_sequence_enrolled`, `hs_email_optout`, `hs_email_optout_<subscriptionId>`.
3. **Plan** (Code node): skip if currently or previously enrolled, or opted out (opt-out properties = true).
4. **Read subscription status** (`GET /communication-preferences/v3/status/email/<email>`) – for logging only.
5. **Set subscription** (`POST /communication-preferences/v3/subscribe`) with `legalBasis` and `legalBasisExplanation` from `configuration.md` (decided by the company); "already subscribed" (400) is harmless → `neverError`.
6. **Enroll**: `POST /automation/v4/sequences/enrollments?userId=<setterUserId>` body `{ sequenceId, contactId, senderEmail }`. Reference data via `$('<name of subscription node>').item.json` (after an HTTP node, `$json` is the response).
7. **Log** the result.

Credentials: admin/private app token with scopes `automation.sequences.enrollments.write`, `communication_preferences.write`, `crm.objects.contacts.read`.
Batching: 1 request / 300–600 ms.
Going live: `mode: 'live'`, publish the workflow, and at the same time remove existing single call tasks from lead workflows (otherwise duplicate call tasks).
