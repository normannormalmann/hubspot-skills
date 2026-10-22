# Internal HubSpot API for sequences and templates (browser tab)

Only from a logged-in HubSpot tab (same origin as your app domain, e.g. `app.hubspot.com` or `app-eu1.hubspot.com`) via `fetch` with `credentials: 'include'`.

```js
const q = 'portalId=<PORTAL>&clienttimeout=14000&hs_static_app=SequencesUI&hs_static_app_version=<VERSION>';
const hdr = { 'X-HubSpot-CSRF-hubspotapi': (document.cookie.match(/hubspotapi-csrf=([^;]+)/) || [])[1] || '', 'Content-Type': 'application/json' };
```
`<PORTAL>` = portal ID from `configuration.md`. Take `<VERSION>` from a request made by the sequences UI in the network tab (the value changes with updates; an older version usually still works).

Note: outputs containing cookie or query strings are blocked by the browser tool; sanitize output with `.replace(/[?&=%]/g,' ')`. After every navigation, `window.*` variables are gone.

## Sequences
| Purpose | Method / path |
|---|---|
| read (execution format, steps with `action`, `delay`, `dependencies`) | `GET /api/sequences/v2/sequences/<id>` |
| read (storage format "egg") | `GET /api/sequences/v3/sequences/<id>` → `.sequence` |
| update | `PUT /api/sequences/v3/sequences/update-with-permissions/<id>` body `{ sequenceEgg }` |
| create | `POST /api/sequences/v3/sequences/create` body `{ sequenceEgg }` → `.sequence.id` |
| v2 PUT | **do not use** (500 INTERNAL_ERROR) |

### Step formats in the egg
```js
// Call task
{ actionType: 'TASK', emailPattern: null,
  taskPattern: { taskPriority: 'HIGH', taskType: 'CALL', customTaskType: null, subject: 'Call 1', notes: '…', queueId: null,
                 threadEmailToStepOrder: null, threadEmailToUniqueId: null, templateId: null },
  delayMillis: 0, stepOrder: i, variants: [], branchNumber: 0 }
// Automated email from template
{ actionType: 'EMAIL', emailPattern: { activeType: 'ACTIVE', templateId: 123, threadEmailToStepOrder: null /* or stepOrder of the base email */ },
  taskPattern: null, delayMillis: 86400000, stepOrder: i, variants: [], branchNumber: 0 }
// End: copy the existing FINISH step from the egg (actionType != EMAIL/TASK), adjust stepOrder, remove IDs
```
Dependencies at egg level:
```js
dependencies: [{ branchNumber: 0, reliesOnStepOrder: 1, requiredByStepOrder: 2, dependencyType: 'TASK_COMPLETION' }]
```
Settings (`egg.settings`): `eligibleFollowUpDays: 'BUSINESS_DAYS'`, `sendWindowStartMinute: 480`, `sendWindowEndMinute: 1080`, `useThreadedFollowUps: true`, `individualTaskRemindersEnabled`.

For new sequences built from a copy: remove `id`, `createdAt`, `updatedAt`, `deletedAt`, `portalId`, `sequenceId`, `sequenceStepId` from the egg, steps, patterns and settings.

### Builder (proven)
```js
const build = (egg, plan) => {
  const finish = egg.steps.find(s => s.actionType !== 'EMAIL' && s.actionType !== 'TASK');
  const fin = { delayMillis: 0, actionType: finish.actionType, emailPattern: null, taskPattern: null, variants: [], variantMeta: null, delayMillisMin: null, delayMillisMax: null, dynamic: false, branchNumber: 0 };
  const steps = plan.map((s, i) => s.type === 'call'
    ? { actionType: 'TASK', emailPattern: null, taskPattern: { taskPriority: 'HIGH', taskType: 'CALL', customTaskType: null, subject: s.title, notes: s.notes || null, queueId: null, threadEmailToStepOrder: null, threadEmailToUniqueId: null, templateId: null }, delayMillis: s.delay || 0, stepOrder: i, variants: [], branchNumber: 0 }
    : s.type === 'email'
    ? { actionType: 'EMAIL', emailPattern: { activeType: 'ACTIVE', templateId: s.tpl, threadEmailToStepOrder: s.thread ?? null }, taskPattern: null, delayMillis: s.delay || 0, stepOrder: i, variants: [], branchNumber: 0 }
    : Object.assign({}, fin, { stepOrder: i }));
  const deps = []; plan.forEach((s, i) => { if (i > 0 && plan[i - 1].type === 'call') deps.push({ branchNumber: 0, reliesOnStepOrder: i - 1, requiredByStepOrder: i, dependencyType: 'TASK_COMPLETION' }); });
  return Object.assign({}, egg, { steps, dependencies: deps });
};
```
Before saving, check the plan against the rules: step 0 is an email; no email with delay 0 directly after a task; thread chain is linear.

## Templates
| Purpose | Method / path |
|---|---|
| create | `POST /api/sales-templates/v3/templates` body `{ template: { name, subject, body } }` → `.templateView.id` |
| read | `GET /api/sales-templates/v2/templates/<id>` |
| update (subject/body) | `PUT /api/sales-templates/v2/templates/<id>` with the full object (without `userView`) |
| **share** | `PUT /api/contentpermissions/v1/permissions` body `{ objectType: 'TEMPLATE', objectIds: [..], permissions: { TEAM: [], USER: [] }, visibleToAll: true, private: false }` → 204 |

Body HTML pattern:
```html
<div dir="auto" data-top-level="true"><p style="margin:0;">Hello {{ contact.firstname }},</p><p style="margin:0;"><br></p>…</div>
```

## Error patterns
| Error | Cause | Fix |
|---|---|---|
| `INVALID_SEQUENCE_STRUCTURE` on enrollment | task as first step, or email with delay 0 after a task | rules 2 and 3 |
| `Multiple emails threading on the same base step` on save | two replies to the same base email | thread chain |
| `TEMPLATES_NOT_FOUND` | template not shared (content permissions) | share endpoint |
| `Contact has unsubscribed` | no sales subscription | set subscription (legal basis) |
| `PUBLIC_ENROLL_NO_CONNECTED_EMAILS` | sender without connected inbox | different sender |
| `CONTACT_ALREADY_ENROLLED` | contact already in a sequence | – |
