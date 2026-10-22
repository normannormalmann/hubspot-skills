# Automatische Einschreibung per n8n

Ablauf (in der Praxis bewährt):

1. **Konfiguration** (Code): `modus` test|live, `testEmails`, `fensterStunden`, Zuordnung Herkunft → Sequenz-ID, Setter-Owner-ID, Setter-User-ID, Absender-Postfach, Abo-ID.
2. **Kandidaten suchen**: `POST /crm/v3/objects/contacts/search`; Filter Owner = Setter, Lead-Status NEW, Herkunft in Zuordnung, E-Mail vorhanden, `num_associated_deals` leer; im Testmodus statt Zeitfenster `email IN testEmails`. Properties inkl. `hs_sequences_is_enrolled`, `hs_latest_sequence_enrolled`, `hs_email_optout`, `hs_email_optout_<aboId>`.
3. **Planen** (Code): überspringen, wenn schon/je in Sequenz oder abgemeldet (Opt-out-Felder = true).
4. **Abo-Status lesen** (`GET /communication-preferences/v3/status/email/<email>`) – nur fürs Protokoll.
5. **Abo setzen** (`POST /communication-preferences/v3/subscribe`) mit `legalBasis` und `legalBasisExplanation` aus `konfiguration.md` (vom Unternehmen festgelegt); „already subscribed" (400) ist harmlos → `neverError`.
6. **Einschreiben**: `POST /automation/v4/sequences/enrollments?userId=<setterUserId>` Body `{ sequenceId, contactId, senderEmail }`. Daten per `$('<Name des Abo-Knotens>').item.json` referenzieren (nach HTTP-Knoten ist `$json` die Antwort).
7. **Ergebnis** protokollieren.

Schlüssel: Admin-/Private-App-Token mit Scopes `automation.sequences.enrollments.write`, `communication_preferences.write`, `crm.objects.contacts.read`.
Batching 1 Anfrage / 300–600 ms.
Live-Schaltung: `modus: 'live'`, Workflow veröffentlichen, parallel bestehende Einzel-Aufgaben in Lead-Workflows entfernen (sonst doppelte Anrufaufgaben).
