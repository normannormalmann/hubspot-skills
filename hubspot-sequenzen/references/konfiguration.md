# Konfiguration (zum Ausfüllen)

Diese Datei enthält die unternehmensspezifischen Werte. Alles in `<…>` ist ein Platzhalter.
Wenn Claude den Skill nutzt und hier noch Platzhalter stehen, ermittelt es die Werte wie jeweils beschrieben und schlägt vor, sie einzutragen.

## Portal

- Portal-ID: `<PORTAL_ID>`
  → steht in jeder HubSpot-URL (`…/contacts/<PORTAL_ID>/…`) oder per `GET /account-info/v3/details`.
- Rechenzentrum / App-Domain: `<app.hubspot.com | app-eu1.hubspot.com>`
  → aus der Browser-URL ablesen. Die internen Endpunkte (siehe `interne-api.md`) nur auf dieser Origin aufrufen.

## Nutzer

| Rolle | Name | User-ID | Owner-ID | Verbundenes Postfach |
|---|---|---|---|---|
| Absender / Setter | `<NAME>` | `<USER_ID>` | `<OWNER_ID>` | `<postfach@firma.de>` |
| Closer (optional) | `<NAME>` | `<USER_ID>` | `<OWNER_ID>` | – |
| Ersteller der Sequenzen | `<NAME>` | `<USER_ID>` | – | `<ja/nein>` |

Ermitteln:
- Owner- und User-IDs: `GET /crm/v3/owners` (Feld `id` = Owner-ID, `userId` = User-ID).
- Verbundenes Postfach: in HubSpot unter Einstellungen → Allgemein → E-Mail des jeweiligen Nutzers prüfen. Nutzer ohne verbundenes Postfach können nicht Absender sein (Regel 7).

## Abos und Rechtsgrundlage (nur DSGVO-aktivierte Portale)

- Sales-/1:1-Abo (z. B. „Direkte Kommunikation" / „One to One"): ID `<ABO_ID_SALES>`
- Marketing-Abo (nur zur Abgrenzung): ID `<ABO_ID_MARKETING>`
  → `GET /communication-preferences/v3/definitions`
- Rechtsgrundlage für die Einschreibung: `<LEGAL_BASIS>`
  Mögliche Werte u. a. `LEGITIMATE_INTEREST_PQL`, `LEGITIMATE_INTEREST_CLIENT`, `PERFORMANCE_OF_CONTRACT`, `CONSENT_WITH_NOTICE`, `NON_GDPR`.
- Begründungstext (`legalBasisExplanation`): `<z. B. „Interessent hat über <Quelle> angefragt; Beantwortung der Anfrage per 1:1-E-Mail und Telefon.">`

**Wichtig:** Welche Rechtsgrundlage gilt, entscheidet das Unternehmen (idealerweise mit Datenschutzbeauftragtem). Claude wählt hier nichts selbst, sondern fragt nach.

## Meeting-Link (optional, für Vorlagen)

- Terminbuchungslink des Absenders: `<https://meetings….hubspot.com/…>`

## Sequenzen

Kurzform: M = Mail, A = Anruf, Zahl = Werktage Verzögerung, * = wartet auf erledigte Aufgabe, >n = Antwort im Thread auf Schritt n, E = Ende.

| Sequenz | ID | Aufbau |
|---|---|---|
| `<Name, z. B. A · Online-Leads>` | `<SEQ_ID>` | `<z. B. M0 A0 A1* A1* M1* A1 A3* M2*>4 E>` |
| `<Name>` | `<SEQ_ID>` | `<…>` |

## Vorlagen (alle freigegeben, siehe Regel 5)

| Kürzel | Zweck | Vorlagen-ID |
|---|---|---|
| `<A1>` | `<Eingangsbestätigung>` | `<TEMPLATE_ID>` |

## Automatisierung (optional)

- n8n-Workflow zur Einschreibung: `<WORKFLOW_ID / Name>`
- HubSpot-Credential in n8n: `<Name des Credentials>`
- Zuordnung Herkunft → Sequenz: `<Wert des Herkunftsfelds>` → `<Sequenz>`; `<…>` → `<…>`
- Herkunftsfeld (interner Name): `<z. B. hs_analytics_source oder eigenes Feld>`
