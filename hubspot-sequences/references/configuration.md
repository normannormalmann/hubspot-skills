# Configuration (to fill in)

This file holds the company-specific values. Everything in `<…>` is a placeholder.
If Claude uses the skill while placeholders remain, it determines the values as described for each item and offers to fill them in.

## Portal

- Portal ID: `<PORTAL_ID>`
  → part of every HubSpot URL (`…/contacts/<PORTAL_ID>/…`) or via `GET /account-info/v3/details`.
- Data center / app domain: `<app.hubspot.com | app-eu1.hubspot.com>`
  → read it from the browser URL. Only call the internal endpoints (see `internal-api.md`) on this origin.

## Users

| Role | Name | User ID | Owner ID | Connected inbox |
|---|---|---|---|---|
| Sender / setter | `<NAME>` | `<USER_ID>` | `<OWNER_ID>` | `<inbox@company.com>` |
| Closer (optional) | `<NAME>` | `<USER_ID>` | `<OWNER_ID>` | – |
| Sequence creator | `<NAME>` | `<USER_ID>` | – | `<yes/no>` |

How to find them:
- Owner and user IDs: `GET /crm/v3/owners` (field `id` = owner ID, `userId` = user ID).
- Connected inbox: in HubSpot under Settings → General → Email for the respective user. Users without a connected inbox cannot be senders (rule 7).

## Subscriptions and legal basis (GDPR-enabled portals only)

- Sales/one-to-one subscription (e.g. "One to One"): ID `<SUBSCRIPTION_ID_SALES>`
- Marketing subscription (for distinction only): ID `<SUBSCRIPTION_ID_MARKETING>`
  → `GET /communication-preferences/v3/definitions`
- Legal basis for enrollment: `<LEGAL_BASIS>`
  Possible values include `LEGITIMATE_INTEREST_PQL`, `LEGITIMATE_INTEREST_CLIENT`, `PERFORMANCE_OF_CONTRACT`, `CONSENT_WITH_NOTICE`, `NON_GDPR`.
- Explanation (`legalBasisExplanation`): `<e.g. "Prospect inquired via <source>; responding to the inquiry by one-to-one email and phone.">`

**Important:** The company decides which legal basis applies (ideally with its data protection officer). Claude never chooses one itself but asks.

## Meeting link (optional, for templates)

- Sender's booking link: `<https://meetings….hubspot.com/…>`

## Sequences

Short form: E = email, C = call, number = business days delay, * = waits for completed task, >n = reply in thread to step n, X = end.

| Sequence | ID | Structure |
|---|---|---|
| `<Name, e.g. A · Online leads>` | `<SEQ_ID>` | `<e.g. E0 C0 C1* C1* E1* C1 C3* E2*>4 X>` |
| `<Name>` | `<SEQ_ID>` | `<…>` |

## Templates (all shared, see rule 5)

| Short name | Purpose | Template ID |
|---|---|---|
| `<A1>` | `<Acknowledgement>` | `<TEMPLATE_ID>` |

## Automation (optional)

- n8n enrollment workflow: `<WORKFLOW_ID / name>`
- HubSpot credential in n8n: `<credential name>`
- Mapping source → sequence: `<value of source property>` → `<sequence>`; `<…>` → `<…>`
- Source property (internal name): `<e.g. hs_analytics_source or a custom property>`
