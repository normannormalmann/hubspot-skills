[English](README.md) · **Deutsch**

# HubSpot-Skills für Claude

Sammlung von [Claude Skills](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview) rund um HubSpot – entstanden aus echter Vertriebs- und Automatisierungsarbeit, nicht aus der Doku.

| Skill | Sprache | Wofür |
|---|---|---|
| [`hubspot-sequenzen`](hubspot-sequenzen/) | Deutsch | HubSpot-Sequenzen (Sales Hub) bauen, ändern, testen und per n8n/API automatisch einschreiben |
| [`hubspot-sequences`](hubspot-sequences/) | English | Derselbe Skill auf Englisch |

Nur eine Sprachversion installieren – beide reagieren auf dieselben Anfragen.

---

## hubspot-sequenzen

Bringt Claude bei, Vertriebskadenzen aus Mails und Anruf-Aufgaben in HubSpot anzulegen – inklusive der Stellen, an denen HubSpot ohne brauchbare Fehlermeldung scheitert.

**Löst unter anderem:**

| Fehler | Ursache in einem Satz |
|---|---|
| `INVALID_SEQUENCE_STRUCTURE` | Sequenz beginnt mit einer Aufgabe oder eine Mail folgt ohne Verzögerung auf eine Aufgabe |
| `TEMPLATES_NOT_FOUND` | Vorlage ist nicht per Content-Freigabe für andere Nutzer sichtbar |
| `Contact has unsubscribed` | DSGVO-Portal: Kontakt hat kein Sales-/1:1-Abo |
| `Multiple emails threading on the same base step` | zwei Mails antworten im Thread auf dieselbe Basis-Mail |
| `PUBLIC_ENROLL_NO_CONNECTED_EMAILS` | Absender hat kein verbundenes Postfach |

**Kann:**
- Kadenz fachlich planen und gegen die harten Regeln prüfen
- Mailvorlagen anlegen und für alle freigeben
- Sequenzen mit Anrufen, Mails, Verzögerungen, „Pausieren bis Aufgabe erledigt" und Antwort-im-Thread schreiben (über die internen Endpunkte aus dem eingeloggten Browser-Tab)
- automatische Einschreibung per n8n inkl. Abo-Setzung, Testmodus und Live-Schaltung aufsetzen

### Beispiel-Kadenz

`M0 A0 A1* A1* M1* A1 A3* M2*>4 A2 A3* E` – 7 Anrufe, 3 Mails, rund 3 Wochen:

| Werktag | Schritt |
|---|---|
| 0 | Mail: Eingangsbestätigung → Anruf 1 |
| 1–2 | Anruf 2, Anruf 3 (jeweils erst nach erledigtem Vorgänger) |
| 3 | Mail: „Nicht erreicht" + Terminlink |
| 4–7 | Anruf 4, Anruf 5 |
| 9 | Mail: Nachfass, als Antwort im selben Thread |
| 11–14 | Anruf 6, Anruf 7 (letzter Versuch) |

Details, Builder-Plan und Vorlagentexte: [`references/beispiel-kadenz.md`](hubspot-sequenzen/references/beispiel-kadenz.md).

### Installation

**Claude.ai / Claude Desktop**
1. Ordner `hubspot-sequenzen` als ZIP packen (oder die `.skill`-Datei aus den Releases laden).
2. In Claude unter Einstellungen → Fähigkeiten (Skills) hochladen.

**Claude Code (Plugin-Marketplace, empfohlen)**
```
/plugin marketplace add normannormalmann/hubspot-skills
/plugin install hubspot-sequenzen@hubspot-skills
```
Neue Skills und Updates kommen per `/plugin marketplace update hubspot-skills`.

**Claude Code (manuell)**
Ordner `hubspot-sequenzen` nach `~/.claude/skills/` (global) oder `.claude/skills/` (Projekt) kopieren.

### Einrichtung

`hubspot-sequenzen/references/konfiguration.md` enthält Platzhalter für Portal-ID, Nutzer, Abo-IDs, Sequenzen und Vorlagen. Du kannst sie selbst ausfüllen – oder Claude ermittelt die Werte beim ersten Einsatz und fragt nach, was es nicht selbst herausfinden kann.

### Voraussetzungen

- HubSpot Sales Hub mit Sequenzen (Professional oder höher)
- Zum Anlegen/Ändern von Sequenzen: eingeloggter HubSpot-Tab, auf den Claude zugreifen kann (z. B. Claude in Chrome)
- Für automatische Einschreibung: Private-App-Token mit den Scopes `automation.sequences.enrollments.write`, `communication_preferences.write`, `crm.objects.contacts.read`; optional n8n

### Hinweise

- Die internen HubSpot-Endpunkte sind undokumentiert und können sich jederzeit ändern. Nutzung auf eigenes Risiko.
- Die Rechtsgrundlage für E-Mail-Abos (DSGVO) muss jedes Unternehmen selbst festlegen. Der Skill gibt keine Rechtsberatung und wählt keine Rechtsgrundlage eigenständig.

---

## Beitragen

Issues und Pull Requests sind willkommen – besonders neue Fehlerbilder und deren Lösung.

## Lizenz

[MIT](LICENSE) © Norman Schönherr
