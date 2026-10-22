---
name: hubspot-sequenzen
description: Bauen, Ändern, Testen und automatisches Einschreiben von HubSpot-Sequenzen (Sales Hub) inkl. Mailvorlagen, Anruf-Aufgaben, Verzögerungen, Pausen bis Aufgabe erledigt, Antwort-im-Thread und DSGVO-Abos. Immer verwenden, wenn es um HubSpot-Sequenzen, Kadenzen, Anruf-/Mail-Abfolgen für Setter oder Vertrieb, Sequenz-Einschreibung per n8n oder API, Sequenz-Vorlagen, Fehler wie INVALID_SEQUENCE_STRUCTURE, TEMPLATES_NOT_FOUND oder UNSUBSCRIBED geht – auch wenn der Nutzer nur „Kadenz", „Nachfass-Abfolge" oder „automatische Anrufaufgaben" sagt.
---

# HubSpot-Sequenzen bauen und automatisch befüllen

Praxiswissen aus dem Aufbau echter Vertriebskadenzen. Die harten Regeln unten stammen aus echten Fehlschlägen; wer sie ignoriert, verliert Stunden.

## Schritt 0: Konfiguration klären

Bevor du etwas anlegst, lies `references/konfiguration.md`.
- Ist die Datei bereits ausgefüllt, nutze die Werte daraus.
- Enthält sie noch Platzhalter (`<…>`), ermittle die Werte wie dort beschrieben (per API oder durch Nachfragen beim Nutzer) und biete an, die Datei auszufüllen. Rate keine IDs.
- Die Rechtsgrundlage für das Sales-Abo (Regel 6) legt **immer der Nutzer bzw. sein Unternehmen** fest – nie selbst wählen, sondern nachfragen.

## Die 7 harten Regeln

1. **Öffentliche API kann Sequenzen nur lesen und einschreiben, nicht anlegen.** Anlegen/Ändern geht nur über die Oberfläche oder die internen Endpunkte aus dem eingeloggten Browser-Tab (z. B. Claude in Chrome, `fetch` mit CSRF-Header). Siehe `references/interne-api.md`.
2. **Bei automatischer Einschreibung (API/n8n) muss Schritt 0 eine Mail sein.** Beginnt die Sequenz mit einer Aufgabe, lehnt `POST /automation/v4/sequences/enrollments` mit `INVALID_SEQUENCE_STRUCTURE` ab. Lösung: erste Mail = Eingangsbestätigung („Danke für Ihre Anfrage, ich melde mich in Kürze telefonisch"), direkt danach Anruf 1 mit Verzögerung 0.
3. **Eine Mail darf nicht mit Verzögerung 0 auf eine Aufgabe folgen** (gleicher Fehler). Mindestens 1 Werktag (`delayMillis: 86400000`) zwischen Aufgabe und nachfolgender Mail.
4. **Antwort im Thread (`threadEmailToStepOrder`) nur als Kette:** Jede Basis-Mail darf genau eine Antwort haben („Multiple emails threading on the same base step"). Kette bauen: M1 ← M2 ← M3, nicht M1 ← M2 und M1 ← M3.
5. **Mailvorlagen brauchen die Content-Freigabe, nicht nur `private:false`.** Sonst `TEMPLATES_NOT_FOUND`, sobald ein anderer Nutzer (z. B. Setter) als Absender einschreibt. Freigabe per `PUT /api/contentpermissions/v1/permissions` mit `visibleToAll: true` (siehe Referenz).
6. **DSGVO-aktivierte Portale: Kontakte brauchen das Sales-/1:1-Abo**, sonst `Contact has unsubscribed`. HubSpot meldet auch für nie gesetzte Abos `NOT_SUBSCRIBED` – echte Abmeldungen daher über die Kontakt-Felder `hs_email_optout` und `hs_email_optout_<aboId>` erkennen, nicht über den Status. Rechtsgrundlage (`legalBasis`) nach Entscheidung des Unternehmens setzen (siehe `references/konfiguration.md`).
7. **Absender braucht verbundenes Postfach.** `senderEmail` muss das in HubSpot verbundene Postfach des `userId` sein; Nutzer ohne Postfach → `PUBLIC_ENROLL_NO_CONNECTED_EMAILS`.

Weitere Fakten:
- „Sequenz pausieren, bis Aufgabe erledigt" = Abhängigkeit `TASK_COMPLETION` (der Folgeschritt `requiredByStepOrder` hängt an `reliesOnStepOrder` der Aufgabe).
- Sequenzen haben keinen An/Aus-Schalter; sie wirken nur auf eingeschriebene Kontakte.
- Ein Kontakt kann nur in einer Sequenz gleichzeitig sein (`CONTACT_ALREADY_ENROLLED`).
- Sequenzen und Vorlagen gehören dem anlegenden Nutzer; der Eigentümer lässt sich per API nicht ändern. Freigabe für alle reicht.
- Token-Format in Vorlagen: `{{ contact.firstname }}`, `{{ contact.lastname }}`, eigene Felder als `{{ contact.<interner_feldname> }}`. Auswahlfelder (z. B. Anrede) als Token vermeiden – es kann der interne Wert erscheinen.
- Die internen Endpunkte sind undokumentiert und können sich ohne Ankündigung ändern. Bei unerwarteten Fehlern zuerst prüfen, ob die Oberfläche selbst noch denselben Aufruf nutzt (Netzwerk-Tab).

## Vorgehen

1. **Kadenz fachlich festlegen** (Tabelle: Schritt, Werktag, Art, Inhalt). Regeln 2–4 gleich berücksichtigen.
2. **Mailvorlagen anlegen** (`POST /api/sales-templates/v3/templates` mit `{template:{name, subject, body}}`), danach **Content-Freigabe** setzen (Regel 5).
3. **Sequenz anlegen**: Am zuverlässigsten erste Version in der Oberfläche anlegen (Automatisierung → Sequenzen → Komplett neu → ein Schritt → Speichern → „Jeder" freigeben), dann per `PUT /api/sequences/v3/sequences/update-with-permissions/<id>` mit vollständigem `sequenceEgg` alle Schritte schreiben. Weitere Sequenzen per `POST /api/sequences/v3/sequences/create` mit `{sequenceEgg}` (Kopie ohne IDs).
4. **Prüfen**: `GET /api/sequences/v2/sequences/<id>` und Schritte als Kurzform ausgeben (M0 A0 A1* …).
5. **Einschreibung testen** mit eigenen Test-Adressen (Plus-Adressen wie `name+t1@domain`), vorher Abo setzen. Bei Strukturfehlern mit Mini-Sequenzen eingrenzen (nur Mail / Mail+Anruf / Anruf+Mail …).
6. **n8n-Einschreibung** im Testmodus, dann live (Muster in `references/n8n-einschreibung.md`).
7. **Ausstieg** regeln: Antwort und gebuchtes Meeting beenden die Sequenz automatisch; weitere Ausstiegsgründe (z. B. Lead-Status disqualifiziert, neuer Deal) per HubSpot-Workflow „Aus Sequenz entfernen".
8. **Aufräumen**: Test-Sequenzen, Test-Kontakte, Test-Aufgaben beim Absender.
9. **Konfiguration nachtragen**: Neue Sequenz-, Vorlagen- und Workflow-IDs in `references/konfiguration.md` festhalten.

## Referenzen

- `references/konfiguration.md` – eigene Portal-, Nutzer-, Abo-, Sequenz- und Vorlagen-IDs (Vorlage zum Ausfüllen)
- `references/interne-api.md` – Endpunkte, Payload-Formate, JS-Snippets für den Browser-Tab
- `references/n8n-einschreibung.md` – Ablauf der n8n-Regel inkl. Abo/Rechtsgrundlage
- `references/beispiel-kadenz.md` – vollständiges Beispiel einer regelkonformen Kadenz mit Builder-Plan
