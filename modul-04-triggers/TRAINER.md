# Modul 04 – Triggers – Erklärungen

Was der Trainer am Beamer erklärt. Die Aufgaben stehen in der [README.md](README.md).

## Ablauf am Beamer (10 min) – Playground links im Workflow

Alle Nodes in **Zone 1 – Node Playground** (Notiz *Playground – Übersicht*). Trigger lassen sich mit **Execute step** einmal testweise auslösen bzw. auf ein Ereignis warten lassen.

| Schritt | Node | Was zeigen |
|---|---|---|
| 1 | **Schedule Trigger (Playground)** | Zeit-basiert: Intervall und Cron. Läuft von selbst erst, wenn der Workflow aktiv ist |
| 2 | **Webhook (Playground)** | Push: Test-URL und Production-URL, Methode. *Execute step* wartet auf einen Aufruf |
| 3 | **On form submission (Playground)** | Push: n8n baut das Formular selbst, Test-URL im Browser öffnen |
| 4 | **Workflow Input Trigger (Playground)** | Push von innen: ein anderer Workflow ruft diesen auf |
| 5 | **Gmail Trigger (Playground)**, **Google Drive Trigger (Playground)**, **Google Sheets Trigger (Playground)** | Polling: n8n fragt regelmäßig nach. Deshalb immer mit Verzögerung |
| 6 | (kein Node) | Chat Trigger gibt es nur einmal pro Workflow, er kommt in Aufgabe 4 |

Faustregel zum Mitnehmen: **Push reagiert sofort, Polling mit Verzögerung.**

Danach: Workflow in `<Name> – Modul 04 – …` umbenennen lassen, dann Zone 2.

---

## Vorab: Dieses Modul ist anders

Trigger verändern keine Daten. Sie **starten** einen Workflow, haben keinen Eingang und erzeugen selbst das erste Item. Deshalb erst die vier Kategorien als Raster erklären (5 min), dann jeden Trigger einmal auslösen. Bei jedem Trigger zwei Fragen: Was kommt raus? Was löst ihn in der Praxis aus?

**Wichtig für die Gruppe:** Ein produktiver Workflow hat genau **einen** Trigger. Hier stehen mehrere in einem Workflow, nur zum Lernen. n8n ist nicht darauf ausgelegt, mehrere Trigger in einem Workflow zu kombinieren und ihre Ergebnisse zu sammeln. Deshalb löst ihr jeden Trigger einzeln und bewusst aus, und jeder trägt sich selbst ins Scoreboard ein.

## Die vier Wege, wie ein Workflow startet

| Kategorie | Wer stößt an? | Beispiele aus der Praxis | Nodes |
|---|---|---|---|
| ⏱️ **Zeit-basiert** | die Uhr | jeden Morgen um 8 ein Report, stündlich eine API abfragen | Schedule Trigger |
| 🖱️ **Manuell** | ein Mensch per Klick in n8n | Testen, Workflows auf Abruf | Manual Trigger (*Execute workflow*). Nur einer pro Workflow, deshalb nicht im Playground. Ihr kennt ihn aus Modul 0 bis 3 |
| 🌐 **Event-basiert (Push)** | ein System oder Mensch schickt etwas an n8n | Formular abgeschickt, Kauf in einem Shop, Chat-Nachricht | Webhook, On form submission, Chat Trigger (Aufgabe 4), Workflow Input Trigger, App-Trigger wie Stripe oder Calendly |
| 🔄 **Polling (Pull)** | n8n fragt selbst regelmäßig nach | jede Minute das Postfach prüfen, neue Zeilen im Sheet | Gmail Trigger, Google Sheets Trigger, Google Drive Trigger, viele weitere App-Trigger |

💡 **Faustregel:** Push reagiert sofort. Polling hat immer eine Verzögerung, höchstens so lang wie das Prüfintervall.

## Node für Node: zeigen, fragen, Fallen

### Schedule Trigger – `Schedule Trigger (Playground)`

**Zeigen:** Intervall-Modi durchklicken (Minuten, Stunden, Tage, Custom/Cron). Cron kurz erklären: `0 8 * * 1` = montags um 8 Uhr. *Execute step*: Der Output enthält den Zeitpunkt.

**Frage:** Jeden Werktag um 9 Uhr ein Bericht, aber nicht am Wochenende. Geht das?
**Antwort:** Ja, mit Cron `0 9 * * 1-5`. Alternative: täglich auslösen und danach per IF den Wochentag prüfen.

**Fallen:**
- Von selbst läuft er nur, wenn der Workflow **aktiv** (veröffentlicht) ist. Im Editor löst man ihn per *Execute step* aus.
- **Zeitzone:** Maßgeblich ist die Zeitzone in den Workflow-Settings bzw. der Instanz. „8 Uhr“ kann sonst 8 Uhr UTC heißen.
- Ein falsch gesetztes Cron-Feld (fünf Felder: Minute, Stunde, Tag, Monat, Wochentag) heißt: Er feuert nie oder selten, ohne Fehlermeldung.

### Webhook – `Webhook (Playground)`

**Zeigen:** *Execute step*, dann die Test-URL im Browser aufrufen (GET). Output erklären: `headers`, `params`, `query`, `body`. Alles aus der Anfrage landet hier.

**Frage:** Webhook oder Gmail Trigger: Was ist schneller?
**Antwort:** Der Webhook reagiert im Moment der Anfrage. Der Gmail Trigger erst beim nächsten Abfragen. Wenn der Dienst Webhooks kann, ist das die schnellere Wahl.

**Fallen:**
- **Test-URL und Production-URL:** Die Test-URL lauscht nur, während im Editor gewartet wird. Die Production-URL nur, wenn der Workflow aktiv ist. Falsche URL beim Absender eingetragen = es kommt nie etwas an.
- Standardmäßig antwortet n8n sofort mit einer leeren Bestätigung. Für eine echte Antwort braucht es *Respond to Webhook*.
- Ein offener Webhook ist für alle erreichbar, die die URL kennen. Produktiv immer absichern (Header Auth oder Basic Auth).

### On form submission – `On form submission (Playground)`

**Zeigen:** Felder anlegen (Text „Name“, Dropdown „Kategorie“), Test-URL öffnen, ausfüllen, absenden. Die Eingaben stehen direkt als Felder im Item.

**Frage:** Wann n8n-Formular, wann Typeform oder Google Forms?
**Antwort:** n8n-Formulare sind in Minuten fertig und brauchen keine Anbindung. Ideal für interne Tools und Prototypen. Für aufwendiges Branding, viel Verzweigung oder sehr viele Nutzer lohnt ein Spezialwerkzeug.

**Fallen:**
- Gleiche Test-/Production-Logik wie beim Webhook.
- Pflichtfelder bewusst setzen. Ein leeres Pflichtfeld verhindert das Absenden.

### Workflow Input Trigger – `Workflow Input Trigger (Playground)`

**Zeigen:** Kein Zeitplan, kein externes Ereignis. Dieser Trigger wartet darauf, dass ein **anderer** Workflow ihn über *Execute Workflow* aufruft. Was der Aufrufer mitgibt, ist hier der Input.

**Frage:** Warum einen Workflow in Sub-Workflows aufteilen?
**Antwort:** Wiederverwenden (mehrere Workflows rufen denselben auf), Übersicht (große Workflows werden unlesbar), Wartung (Teile einzeln testen und ändern).

**Fallen:**
- Welche Felder ankommen, legt der Trigger fest (*Input data mode*). Was nicht definiert oder nicht übergeben wird, fehlt.
- Ein Fehler im Sub-Workflow lässt auch den Aufrufer scheitern, wenn er nicht abgefangen wird.

### Gmail Trigger – `Gmail Trigger (Playground)`

**Zeigen:** Abfrageintervall (Standard: jede Minute), Filter nach Absender, Betreff, Label. Testmail schicken, *Execute step*: Betreff, Absender, Text, Anhänge.

**Frage:** Eine eilige Bestellung kommt per Mail. Ist der Gmail Trigger richtig?
**Antwort:** Meist ja: Bei „jede Minute“ höchstens 60 Sekunden Verzögerung. Wenn es sofort sein muss, braucht es einen Push-Weg (z. B. Gmail über Google Pub/Sub an einen Webhook). Der ist aufwendiger.

**Fallen:**
- Er liefert nur Mails, die seit dem letzten Abruf neu sind.
- Mehrere Google-Konten: In der Credential das richtige auswählen.

### Google Sheets Trigger – `Google Sheets Trigger (Playground)`

**Zeigen:** Sheet und Tabellenblatt wählen, Ereignis wählen (*Row Added*, *Row Updated* oder beides). Zeile eintragen, *Execute step*.

**Frage:** Was ist der Unterschied zum normalen Google-Sheets-Node, der Zeilen liest?
**Antwort:** Der Node liest einmal, wenn er an der Reihe ist. Der Trigger beobachtet das Sheet und startet den Workflow selbst. Trigger = reagiert, Node = auf Anforderung.

**Fallen:**
- Zeile 1 muss die Kopfzeile sein, sonst stimmt die Zuordnung der Felder nicht.
- Polling: Verzögerung bis zum nächsten Abruf.

### Google Drive Trigger – `Google Drive Trigger (Playground)`

**Zeigen:** Ordner wählen, Ereignis wählen (Datei neu, Datei geändert). Testdatei hochladen, *Execute step*: Name, ID, Dateityp.

**Frage:** Wofür würdet ihr ihn einsetzen?
**Antwort:** Rechnungen oder Verträge verarbeiten, sobald sie im Ordner liegen. Bilder beim Hochladen umbenennen oder umwandeln. Neue Dateien ins CRM eintragen. Modul 6 zeigt, warum dort trotzdem ein Schedule Trigger mit *List Files* genommen wird.

**Falle:** Polling, also nichts für zeitkritische Abläufe.

### Übergang zur Challenge

„Ihr habt alle vier Kategorien live gesehen, von der Uhr über Push bis Polling. In der Challenge löst ihr jeden Trigger selbst aus, auch den Chat Trigger. Ich starte die Zeit, wenn alle bereit sind.“
