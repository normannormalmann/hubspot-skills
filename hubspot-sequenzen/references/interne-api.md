# Interne HubSpot-API für Sequenzen und Vorlagen (Browser-Tab)

Nur aus einem eingeloggten HubSpot-Tab (gleiche Origin wie die eigene App-Domain, z. B. `app.hubspot.com` oder `app-eu1.hubspot.com`) per `fetch` mit `credentials: 'include'`.

```js
const q = 'portalId=<PORTAL>&clienttimeout=14000&hs_static_app=SequencesUI&hs_static_app_version=<VERSION>';
const hdr = { 'X-HubSpot-CSRF-hubspotapi': (document.cookie.match(/hubspotapi-csrf=([^;]+)/) || [])[1] || '', 'Content-Type': 'application/json' };
```
`<PORTAL>` = Portal-ID aus `konfiguration.md`. `<VERSION>` aus einem Aufruf der Sequenzen-Oberfläche im Netzwerk-Tab übernehmen (der Wert ändert sich mit Updates; meist funktioniert auch eine ältere Version).

Hinweis: Ausgaben, die Cookie- oder Query-Strings enthalten, werden vom Browser-Tool blockiert; Ausgabe mit `.replace(/[?&=%]/g,' ')` säubern. Nach jeder Navigation sind `window.*`-Variablen weg.

## Sequenzen
| Zweck | Methode / Pfad |
|---|---|
| lesen (Ausführungsformat, Schritte mit `action`, `delay`, `dependencies`) | `GET /api/sequences/v2/sequences/<id>` |
| lesen (Speicherformat „Egg") | `GET /api/sequences/v3/sequences/<id>` → `.sequence` |
| ändern | `PUT /api/sequences/v3/sequences/update-with-permissions/<id>` Body `{ sequenceEgg }` |
| neu | `POST /api/sequences/v3/sequences/create` Body `{ sequenceEgg }` → `.sequence.id` |
| v2-PUT | **nicht verwenden** (500 INTERNAL_ERROR) |

### Schritt-Formate im Egg
```js
// Anruf-Aufgabe
{ actionType: 'TASK', emailPattern: null,
  taskPattern: { taskPriority: 'HIGH', taskType: 'CALL', customTaskType: null, subject: 'Anruf 1', notes: '…', queueId: null,
                 threadEmailToStepOrder: null, threadEmailToUniqueId: null, templateId: null },
  delayMillis: 0, stepOrder: i, variants: [], branchNumber: 0 }
// Automatische Mail aus Vorlage
{ actionType: 'EMAIL', emailPattern: { activeType: 'ACTIVE', templateId: 123, threadEmailToStepOrder: null /* oder Order der Basis-Mail */ },
  taskPattern: null, delayMillis: 86400000, stepOrder: i, variants: [], branchNumber: 0 }
// Ende: den vorhandenen FINISH-Schritt des Eggs kopieren (actionType != EMAIL/TASK), stepOrder anpassen, IDs entfernen
```
Abhängigkeiten auf Egg-Ebene:
```js
dependencies: [{ branchNumber: 0, reliesOnStepOrder: 1, requiredByStepOrder: 2, dependencyType: 'TASK_COMPLETION' }]
```
Einstellungen (`egg.settings`): `eligibleFollowUpDays: 'BUSINESS_DAYS'`, `sendWindowStartMinute: 480`, `sendWindowEndMinute: 1080`, `useThreadedFollowUps: true`, `individualTaskRemindersEnabled`.

Für neue Sequenzen aus einer Kopie: `id`, `createdAt`, `updatedAt`, `deletedAt`, `portalId`, `sequenceId`, `sequenceStepId` in Egg, Schritten, Patterns und Settings entfernen.

### Builder (bewährt)
```js
const bau = (egg, plan) => {
  const finish = egg.steps.find(s => s.actionType !== 'EMAIL' && s.actionType !== 'TASK');
  const fin = { delayMillis: 0, actionType: finish.actionType, emailPattern: null, taskPattern: null, variants: [], variantMeta: null, delayMillisMin: null, delayMillisMax: null, dynamic: false, branchNumber: 0 };
  const steps = plan.map((s, i) => s.typ === 'call'
    ? { actionType: 'TASK', emailPattern: null, taskPattern: { taskPriority: 'HIGH', taskType: 'CALL', customTaskType: null, subject: s.titel, notes: s.notes || null, queueId: null, threadEmailToStepOrder: null, threadEmailToUniqueId: null, templateId: null }, delayMillis: s.delay || 0, stepOrder: i, variants: [], branchNumber: 0 }
    : s.typ === 'mail'
    ? { actionType: 'EMAIL', emailPattern: { activeType: 'ACTIVE', templateId: s.tpl, threadEmailToStepOrder: s.thread ?? null }, taskPattern: null, delayMillis: s.delay || 0, stepOrder: i, variants: [], branchNumber: 0 }
    : Object.assign({}, fin, { stepOrder: i }));
  const deps = []; plan.forEach((s, i) => { if (i > 0 && plan[i - 1].typ === 'call') deps.push({ branchNumber: 0, reliesOnStepOrder: i - 1, requiredByStepOrder: i, dependencyType: 'TASK_COMPLETION' }); });
  return Object.assign({}, egg, { steps, dependencies: deps });
};
```
Vor dem Speichern Plan gegen die Regeln prüfen: Schritt 0 Mail; keine Mail mit delay 0 direkt nach Aufgabe; Thread-Kette linear.

## Vorlagen
| Zweck | Methode / Pfad |
|---|---|
| neu | `POST /api/sales-templates/v3/templates` Body `{ template: { name, subject, body } }` → `.templateView.id` |
| lesen | `GET /api/sales-templates/v2/templates/<id>` |
| ändern (Betreff/Text) | `PUT /api/sales-templates/v2/templates/<id>` mit vollem Objekt (ohne `userView`) |
| **freigeben** | `PUT /api/contentpermissions/v1/permissions` Body `{ objectType: 'TEMPLATE', objectIds: [..], permissions: { TEAM: [], USER: [] }, visibleToAll: true, private: false }` → 204 |

Body-HTML-Muster:
```html
<div dir="auto" data-top-level="true"><p style="margin:0;">Guten Tag {{ contact.firstname }} {{ contact.lastname }},</p><p style="margin:0;"><br></p>…</div>
```

## Fehlerbilder
| Fehler | Ursache | Lösung |
|---|---|---|
| `INVALID_SEQUENCE_STRUCTURE` beim Einschreiben | Aufgabe als erster Schritt, oder Mail mit delay 0 nach Aufgabe | Regeln 2 und 3 |
| `Multiple emails threading on the same base step` beim Speichern | zwei Antworten auf dieselbe Basis | Thread-Kette |
| `TEMPLATES_NOT_FOUND` | Vorlage nicht freigegeben (Content-Permissions) | Freigabe-Endpunkt |
| `Contact has unsubscribed` | kein Sales-Abo | Abo setzen (Rechtsgrundlage) |
| `PUBLIC_ENROLL_NO_CONNECTED_EMAILS` | Absender ohne verbundenes Postfach | anderen Absender |
| `CONTACT_ALREADY_ENROLLED` | Kontakt schon in einer Sequenz | – |
