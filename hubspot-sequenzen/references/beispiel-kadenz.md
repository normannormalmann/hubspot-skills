# Beispiel: regelkonforme Kadenz für Inbound-Leads

Eine Kadenz für Interessenten, die selbst angefragt haben (z. B. Formular oder Portal-Anfrage): 7 Anrufe, 3 Mails, rund 3 Wochen. Sie erfüllt alle harten Regeln aus `SKILL.md` und kann per API/n8n eingeschrieben werden.

Kurzform: `M0 A0 A1* A1* M1* A1 A3* M2*>4 A2 A3* E`
(M = Mail, A = Anruf, Zahl = Werktage Verzögerung, * = wartet auf erledigte Aufgabe, >n = Antwort im Thread auf Schritt n, E = Ende)

| stepOrder | Werktag (ca.) | Art | Verzögerung | Inhalt | Warum so |
|---|---|---|---|---|---|
| 0 | 0 | Mail | 0 | Eingangsbestätigung: „Danke für Ihre Anfrage, ich melde mich in Kürze telefonisch." | Regel 2: Schritt 0 muss Mail sein |
| 1 | 0 | Anruf | 0 | Anruf 1 – Erstkontakt | sofort nach Bestätigung |
| 2 | 1 | Anruf | 1 | Anruf 2 | wartet, bis Anruf 1 erledigt |
| 3 | 2 | Anruf | 1 | Anruf 3 | wartet, bis Anruf 2 erledigt |
| 4 | 3 | Mail | 1 | „Habe Sie telefonisch nicht erreicht" + Terminlink | Regel 3: ≥ 1 Werktag nach Aufgabe |
| 5 | 4 | Anruf | 1 | Anruf 4 | |
| 6 | 7 | Anruf | 3 | Anruf 5 | wartet, bis Anruf 4 erledigt |
| 7 | 9 | Mail | 2 | Nachfass-Mail, als Antwort auf Schritt 4 | Regel 4: einzige Antwort auf Basis 4 |
| 8 | 11 | Anruf | 2 | Anruf 6 | |
| 9 | 14 | Anruf | 3 | Anruf 7 – letzter Versuch | wartet, bis Anruf 6 erledigt |
| 10 | – | Ende | 0 | – | |

Verzögerungen gelten jeweils ab dem vorherigen Schritt bzw. ab Erledigung der Aufgabe, daher sind die Werktage nur Richtwerte.

## Plan für den Builder (`references/interne-api.md`)

```js
const T = 86400000; // 1 Werktag
const plan = [
  { typ: 'mail', tpl: <VORLAGE_EINGANG>, delay: 0 },
  { typ: 'call', titel: 'Anruf 1 – Erstkontakt', delay: 0 },
  { typ: 'call', titel: 'Anruf 2', delay: 1 * T },
  { typ: 'call', titel: 'Anruf 3', delay: 1 * T },
  { typ: 'mail', tpl: <VORLAGE_NICHT_ERREICHT>, delay: 1 * T },
  { typ: 'call', titel: 'Anruf 4', delay: 1 * T },
  { typ: 'call', titel: 'Anruf 5', delay: 3 * T },
  { typ: 'mail', tpl: <VORLAGE_NACHFASS>, delay: 2 * T, thread: 4 },
  { typ: 'call', titel: 'Anruf 6', delay: 2 * T },
  { typ: 'call', titel: 'Anruf 7 – letzter Versuch', delay: 3 * T },
  { typ: 'ende' }
];
// const egg = (await (await fetch(`/api/sequences/v3/sequences/${id}?${q}`, { credentials: 'include' })).json()).sequence;
// await fetch(`/api/sequences/v3/sequences/update-with-permissions/${id}?${q}`, { method: 'PUT', headers: hdr, credentials: 'include', body: JSON.stringify({ sequenceEgg: bau(egg, plan) }) });
```

Der Builder setzt die `TASK_COMPLETION`-Abhängigkeit automatisch für jeden Schritt, der direkt auf einen Anruf folgt (in der Kurzform mit `*` markiert).

## Beispiel-Vorlagen (Platzhalter anpassen)

**Eingangsbestätigung**
Betreff: `Ihre Anfrage bei <FIRMA>`
> Guten Tag {{ contact.firstname }} {{ contact.lastname }},
> vielen Dank für Ihre Anfrage. Ich melde mich in Kürze telefonisch bei Ihnen.
> Viele Grüße
> <ABSENDER>

**Nicht erreicht**
Betreff: `Kurzer Rückruf zu Ihrer Anfrage`
> Guten Tag {{ contact.firstname }} {{ contact.lastname }},
> ich habe versucht, Sie telefonisch zu erreichen. Gern können Sie sich hier direkt einen passenden Termin aussuchen: <TERMINLINK>
> Viele Grüße
> <ABSENDER>

**Nachfass (Antwort im Thread auf „Nicht erreicht")**
> Guten Tag {{ contact.firstname }} {{ contact.lastname }},
> ich wollte noch einmal nachhaken – ist Ihre Anfrage noch aktuell? Ein kurzes „Ja" oder „Nein" genügt.
> Viele Grüße
> <ABSENDER>
