# Example: rule-compliant cadence for inbound leads

A cadence for prospects who inquired themselves (e.g. form or portal inquiry): 7 calls, 3 emails, about 3 weeks. It satisfies all hard rules from `SKILL.md` and can be enrolled via API/n8n.

Short form: `E0 C0 C1* C1* E1* C1 C3* E2*>4 C2 C3* X`
(E = email, C = call, number = business days delay, * = waits for completed task, >n = reply in thread to step n, X = end)

| stepOrder | Business day (approx.) | Type | Delay | Content | Why |
|---|---|---|---|---|---|
| 0 | 0 | Email | 0 | Acknowledgement: "Thanks for your inquiry, I'll call you shortly." | Rule 2: step 0 must be an email |
| 1 | 0 | Call | 0 | Call 1 – first contact | right after the acknowledgement |
| 2 | 1 | Call | 1 | Call 2 | waits until call 1 is completed |
| 3 | 2 | Call | 1 | Call 3 | waits until call 2 is completed |
| 4 | 3 | Email | 1 | "Couldn't reach you by phone" + booking link | Rule 3: ≥ 1 business day after a task |
| 5 | 4 | Call | 1 | Call 4 | |
| 6 | 7 | Call | 3 | Call 5 | waits until call 4 is completed |
| 7 | 9 | Email | 2 | Follow-up email, as reply to step 4 | Rule 4: only reply to base 4 |
| 8 | 11 | Call | 2 | Call 6 | |
| 9 | 14 | Call | 3 | Call 7 – last attempt | waits until call 6 is completed |
| 10 | – | End | 0 | – | |

Delays count from the previous step or from task completion, so the business days are approximate.

## Plan for the builder (`references/internal-api.md`)

```js
const D = 86400000; // 1 business day
const plan = [
  { type: 'email', tpl: <TEMPLATE_ACK>, delay: 0 },
  { type: 'call', title: 'Call 1 – first contact', delay: 0 },
  { type: 'call', title: 'Call 2', delay: 1 * D },
  { type: 'call', title: 'Call 3', delay: 1 * D },
  { type: 'email', tpl: <TEMPLATE_NOT_REACHED>, delay: 1 * D },
  { type: 'call', title: 'Call 4', delay: 1 * D },
  { type: 'call', title: 'Call 5', delay: 3 * D },
  { type: 'email', tpl: <TEMPLATE_FOLLOW_UP>, delay: 2 * D, thread: 4 },
  { type: 'call', title: 'Call 6', delay: 2 * D },
  { type: 'call', title: 'Call 7 – last attempt', delay: 3 * D },
  { type: 'end' }
];
// const egg = (await (await fetch(`/api/sequences/v3/sequences/${id}?${q}`, { credentials: 'include' })).json()).sequence;
// await fetch(`/api/sequences/v3/sequences/update-with-permissions/${id}?${q}`, { method: 'PUT', headers: hdr, credentials: 'include', body: JSON.stringify({ sequenceEgg: build(egg, plan) }) });
```

The builder automatically adds the `TASK_COMPLETION` dependency for every step that directly follows a call (marked with `*` in the short form).

## Example templates (adjust placeholders)

**Acknowledgement**
Subject: `Your inquiry at <COMPANY>`
> Hello {{ contact.firstname }},
> thank you for your inquiry. I'll give you a call shortly.
> Best regards
> <SENDER>

**Couldn't reach you**
Subject: `Quick follow-up on your inquiry`
> Hello {{ contact.firstname }},
> I tried to reach you by phone. Feel free to pick a time that suits you here: <BOOKING_LINK>
> Best regards
> <SENDER>

**Follow-up (reply in thread to "Couldn't reach you")**
> Hello {{ contact.firstname }},
> just checking in – is your inquiry still relevant? A quick "yes" or "no" is enough.
> Best regards
> <SENDER>
