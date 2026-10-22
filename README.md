**English** · [Deutsch](README.de.md)

# HubSpot Skills for Claude

A collection of [Claude Skills](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview) for HubSpot – built from real sales and automation work, not from the docs.

| Skill | Language | Purpose |
|---|---|---|
| [`hubspot-sequences`](hubspot-sequences/) | English | Build, edit, test and auto-enroll HubSpot sequences (Sales Hub) via n8n/API |
| [`hubspot-sequenzen`](hubspot-sequenzen/) | Deutsch | Same skill in German |

Install only one language version – both trigger on the same requests.

---

## hubspot-sequences

Teaches Claude to build sales cadences of emails and call tasks in HubSpot – including the spots where HubSpot fails without a useful error message.

**Solves, among others:**

| Error | Cause in one sentence |
|---|---|
| `INVALID_SEQUENCE_STRUCTURE` | Sequence starts with a task, or an email follows a task without delay |
| `TEMPLATES_NOT_FOUND` | Template isn't shared with other users via content permissions |
| `Contact has unsubscribed` | GDPR portal: contact has no sales/one-to-one subscription |
| `Multiple emails threading on the same base step` | Two emails reply in thread to the same base email |
| `PUBLIC_ENROLL_NO_CONNECTED_EMAILS` | Sender has no connected inbox |

**Can:**
- plan a cadence and check it against the hard rules
- create email templates and share them with everyone
- write sequences with calls, emails, delays, "pause until task is completed" and reply-in-thread (via the internal endpoints from a logged-in browser tab)
- set up automatic enrollment via n8n incl. subscription handling, test mode and go-live

### Example cadence

`E0 C0 C1* C1* E1* C1 C3* E2*>4 C2 C3* X` – 7 calls, 3 emails, about 3 weeks:

| Business day | Step |
|---|---|
| 0 | Email: acknowledgement → call 1 |
| 1–2 | Call 2, call 3 (each only after the previous one is completed) |
| 3 | Email: "Couldn't reach you" + booking link |
| 4–7 | Call 4, call 5 |
| 9 | Email: follow-up, as reply in the same thread |
| 11–14 | Call 6, call 7 (last attempt) |

Details, builder plan and template texts: [`references/example-cadence.md`](hubspot-sequences/references/example-cadence.md).

### Installation

**Claude.ai / Claude Desktop**
1. Zip the `hubspot-sequences` folder (or download the `.skill` file from Releases).
2. Upload it in Claude under Settings → Capabilities (Skills).

**Claude Code (plugin marketplace, recommended)**
```
/plugin marketplace add normannormalmann/hubspot-skills
/plugin install hubspot-sequences@hubspot-skills
```
New skills and updates arrive via `/plugin marketplace update hubspot-skills`.

**Claude Code (manual)**
Copy the `hubspot-sequences` folder to `~/.claude/skills/` (global) or `.claude/skills/` (project).

### Setup

`hubspot-sequences/references/configuration.md` contains placeholders for portal ID, users, subscription IDs, sequences and templates. Fill them in yourself – or let Claude determine the values on first use and ask for whatever it can't find out itself.

### Requirements

- HubSpot Sales Hub with sequences (Professional or higher)
- To create/edit sequences: a logged-in HubSpot tab Claude can access (e.g. Claude in Chrome)
- For automatic enrollment: private app token with scopes `automation.sequences.enrollments.write`, `communication_preferences.write`, `crm.objects.contacts.read`; optionally n8n

### Notes

- The internal HubSpot endpoints are undocumented and may change at any time. Use at your own risk.
- Each company has to decide the legal basis for email subscriptions (GDPR) itself. The skill provides no legal advice and never picks a legal basis on its own.

---

## Contributing

Issues and pull requests are welcome – especially new error patterns and their fixes.

## License

[MIT](LICENSE) © Norman Schönherr
