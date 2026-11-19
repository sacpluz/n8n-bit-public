# Modul 07 – AI + Model

Dieses Handout ist der Text der Sticky Notes aus dem Workflow, damit ihr ihn im zweiten Browser-Tab lesen und Code kopieren könnt.

**Workflow-Datei:** [`modul-07-ai-model.json`](modul-07-ai-model.json)

**Erklärungen zum Modul** (was der Trainer am Beamer zeigt, zum Nachlesen): [TRAINER.md](TRAINER.md)

---

Automatisierter Support mit AI

| ⏱ Dauer | ca. 20 Minuten | 10 Nodes | 👥 Teamgröße 2 Personen |

---

## 🎯 Szenario

Euer Unternehmen bekommt täglich Nachrichten von Kunden — Rechnungsfragen, technische Probleme, Feedback. Bisher landet alles im selben Postfach und wird manuell sortiert. Ihr baut einen Workflow, der eingehende Chat-Nachrichten automatisch nach Stimmung filtert, nach Thema klassifiziert und jeweils mit einer passenden KI-Antwort reagiert. Rechnungsanfragen werden zusätzlich mit echten Kundendaten aus der Datenbank beantwortet.

Der Workflow ist bereits vollständig aufgebaut und verdrahtet. Eure Aufgabe: die einzelnen Nodes konfigurieren, sodass der Flow korrekt funktioniert.

---

## Workflow-Übersicht

```
Chat-Nachricht
  → Sentiment Analysis
      → [negativ]  Stop and Error
      → [sonst]    Text Classifier
                     → Rechnung          → Information Extractor → GetCustomer → Invoice Answer
                     → Tech. Problem     → Evaluate Problem → Technical Support Answer
                     → Feedback          → Answer Feedback
```

---

## Aufgaben

### 1. Sentiment Analysis – Eingabe konfigurieren (`Sentiment Analysis`)
- [ ] Setze das **Input**-Feld auf: `{{ $json.chatInput }}`
- [ ] Technisches Feedback wertet die KI als Negativ, weshalb ein **System Prompt Template** gesetzt werden muss: `Analyze the sentiment based on the tone and language of the message, not the topic. A neutral description of a technical problem should be classified as neutral, even if the topic is negative.`
- [ ] Optional: Aktiviere unter **Add Option** die Einstellung **Include Detailed Results**.

> 💡 **Hinweis:** Der Node hat drei Ausgänge: positiv, neutral, negativ. Der negative Ausgang führt direkt zu *Stop and Error* — unhöfliche Nachrichten brechen den Workflow ab.

---

### 2. Text Classifier – Kategorien anlegen (`Text Classifier`)
- [ ] Füge folgende **Categories** hinzu:
  - `Rechnung`
  - `technisches Problem`
  - `Feedback`
- [ ] Optional: Füge kurze **Descriptions** hinzu, um die Klassifizierung zu präzisieren.

> 💡 **Hinweis:** Die Ausgänge entsprechen der Reihenfolge der Kategorien — Ausgang 0 = Rechnung, 1 = technisches Problem, 2 = Feedback.

---

### 3. Information Extractor LLM Chain – Felder definieren (`Information Extractor`)
- [ ] Füge folgende **Attributes** hinzu:

  | Name | Description |
  |---|---|
  | `Name` | Name des Kunden im Format Vorname Nachname — kann fehlen |
  | `Kundennummer` | Kundennummer im Format CUST-001 — kann fehlen |
  | `Rechnungsnummer` | Rechnungsnummer im Format INV-001 — kann fehlen |

> 💡 **Hinweis:** Alle drei Felder sind optional — Kunden nennen manchmal nur ihren Namen, manchmal nur eine Nummer. Der Output landet unter `$json.output.Name`, `$json.output.Kundennummer` und `$json.output.Rechnungsnummer`.

---

#### 3.1 GetCustomer – Tabellenfilter setzen (`GetCustomer`)

| Einstellung | Wert |
|---|---|
| Resource | Row |
| Operation | Get |
| Must Match | Any Condition |

Filter:

| # | Feld | Wert |
|---|---|---|
| 1 | `fullname` | `{{ $json.output.Name }}` |
| 2 | `id` | `{{ $json.output.Kundennummer }}` |
| 3 | `invoice_nr` | `{{ $json.output.Rechnungsnummer }}` |

---

#### 3.2 Invoice Answer Chain

| Einstellung | Wert |
|---|---|
| Source for Prompt (User Message) | `Define below` |
| Prompt (User Message) | siehe unten |

````
Folgende Rechnungsdaten sind in der Datenbank bekannt:    
Kundennummer: {{ $json.nr }}
Rechnungsdatum: {{ $json.date }}
Bezahlstatus: {{ $json.status }}
Betrag: {{ $json.amount }} Euro
Produkt: {{ $json.product }}    
Beantworte seine ursprüngliche Frage:    
{{ $('When chat message received').item.json.chatInput }}
````

Unter **Chat Messages (if Using a Chat Model)** eine Nachricht anlegen:

| Type Name or ID | Message |
|---|---|
| System | `Du bist ein freundlicher Support-Mitarbeiter. Du beantwortest Rechnungsfragen ausschließlich auf Basis der Kundendaten die dir übergeben werden. Erfinde keine Daten. Wenn kein passender Datensatz gefunden wurde, bitte den Kunden höflich um mehr Informationen.` |

---

### 4. Evaluate Problem LLM Chain

Unter **Chat Messages (if Using a Chat Model)** eine Nachricht anlegen:

| Type Name or ID | Message |
|---|---|
| System | `Sie sind ein technischer Experte. Ermitteln Sie auf Basis der Benutzereingabe die wahrscheinlichsten technischen Probleme sowie entsprechende Lösungsansätze. Liefern Sie eine tiefgreifende technische Analyse für Ihren Kollegen im First-Level-Support. Antworten Sie nicht direkt an den Benutzer, sondern konzentrieren Sie sich ausschließlich auf das technische Problem.` |

#### 4.1 Technical Support Answer LLM Chain
Referenziert die Antwort des Nodes Evaluate Problem:

| Einstellung | Wert |
|---|---|
| Source for Prompt (User Message) | `Define below` |
| Prompt (User Message) | `{{ $('When chat message received').item.json.chatInput }}` |

Unter **Chat Messages (if Using a Chat Model)**:

| Type Name or ID | Message |
|---|---|
| System | `Sie sind Mitarbeiter im First-Level-Support. Sie sind dafür verantwortlich, dem Benutzer zu antworten. Ihr Mitarbeiter aus dem Third-Level-Deep-Tech-Support hat das Problem bereits eingehend analysiert. Hier sind seine Vorschläge zur Anfrage des Benutzers: {{ $json.text }}` |

### 5 Answer Feedback LLM Chain
Unter **Chat Messages (if Using a Chat Model)**:

| Type Name or ID | Message |
|---|---|
| System | `Du bist Support-Mitarbeiter bei bIT. Du weißt, dass Feedback wichtig ist, und würdigst es mit einer kurzen Rückmeldung. Bei bIT sind wir alle eine Familie. Antworte zügig – einfach, um den Leuten zu zeigen, dass du dankbar bist.` |

---

### 6. Workflow testen
- [ ] Schickt mindestens **vier Testnachrichten** im Chat aus den Chat Beispielen und überprüfe was die Agenten jeweils Antworten.

---

## 💡 Tipps

- **Sentiment Analysis testen**: Schickt zuerst eine neutrale Nachricht — so seht ihr den Output-Aufbau bevor ihr weiter konfiguriert. Manchmal sind technische Probleme negativ :D
- **Execute Step**: Jeden Node einzeln ausführen bevor ihr weiterbaut. So seht ihr sofort ob die Konfiguration stimmt.
- **Information Extractor Output**: Schaut euch den Output genau an — `$json.output.Name`, `$json.output.Kundennummer` und `$json.output.Rechnungsnummer` müssen exakt so referenziert werden.
- **Kein passender Datensatz?**: Wenn GetCustomer keine Daten liefert, prüft ob Name und Kundennummer exakt mit der Tabelle übereinstimmen.

---

## Tabellendaten (Invoice)

| nr | invoice_nr | fullname | email | date | status | amount | product |
|---|---|---|---|---|---|---|---|
| CUST-001 | INV-001 | sabine müller | sabine.mueller@example.com | 2025-01-14 | bezahlt | 1.249,00 € | Consulting Paket M |
| CUST-002 | INV-002 | tom berger | tom.berger@example.com | 2025-02-03 | offen | 380,00 € | Support Flatrate |
| CUST-003 | INV-003 | lisa schmitt | lisa.schmitt@example.com | 2025-01-28 | bezahlt | 2.490,00 € | Consulting Paket L |
| CUST-004 | INV-004 | jan richter | jan.richter@example.com | 2025-03-11 | überfällig | 380,00 € | Support Flatrate |
| CUST-005 | INV-005 | petra klein | petra.klein@example.com | 2025-02-19 | bezahlt | 1.249,00 € | Consulting Paket M |
| CUST-006 | INV-006 | markus weber | markus.weber@example.com | 2025-03-05 | offen | 2.490,00 € | Consulting Paket L |
| CUST-007 | INV-007 | anna fischer | anna.fischer@example.com | 2025-01-07 | bezahlt | 380,00 € | Support Flatrate |
| CUST-008 | INV-008 | david schröder | david.schroeder@example.com | 2025-02-25 | überfällig | 1.249,00 € | Consulting Paket M |
| CUST-009 | INV-009 | julia braun | julia.braun@example.com | 2025-03-18 | offen | 2.490,00 € | Consulting Paket L |
| CUST-010 | INV-010 | michael wolf | michael.wolf@example.com | 2025-03-01 | bezahlt | 1.249,00 € | Consulting Paket M |

---

# Chat-Beispiele – Automatisierter Support mit AI

---

## Rechnung

1. *„Hallo, ich bin Sabine Müller. Ich wollte fragen ob meine Rechnung schon bezahlt wurde?"*
2. *„Guten Tag, ich bin Kunde CUST-004. Was ist der aktuelle Status meiner Rechnung?"*
3. *„Hi, ich habe die Rechnungsnummer INV-009 — können Sie mir sagen welches Produkt ich damals gekauft habe?"*
4. *„Hallo, ich bin Markus Weber, Kundennummer CUST-006, Rechnungsnummer INV-006. Ist meine Rechnung schon bezahlt?"*

---

## Technisches Problem

1. *„Seit dem letzten Update lässt sich das Dashboard nicht mehr öffnen. Ich bekomme nur einen weißen Bildschirm."*
2. *„Meine Exportfunktion hängt sich immer auf wenn ich mehr als 100 Datensätze auswähle. Das passiert seit gestern."*
3. *„Ich kann mich nicht mehr einloggen — das Passwort ist korrekt aber nach dem Klick auf 'Anmelden' passiert nichts."*

---

## Feedback

### Positiv

1. *„Ich wollte mich kurz melden — euer Support-Team hat mein Problem gestern super schnell gelöst. Danke!"*
2. *„Die neue Oberfläche ist wirklich gelungen, viel übersichtlicher als vorher. Weiter so!"*
3. *„Ich bin sehr zufrieden mit dem Consulting Paket. Die Einrichtung hat reibungslos geklappt."*

### Neutral

1. *„Ich nutze den Service jetzt seit drei Monaten. Läuft soweit, nichts zu beanstanden."*
2. *„Die Antwortzeiten sind okay. Manchmal dauert es etwas länger, aber es klappt."*
3. *„Das Onboarding war in Ordnung. Ein bisschen mehr Dokumentation wäre hilfreich gewesen."*

### Negativ

1. *„Das ist doch totaler Mist hier, nichts funktioniert wie es soll!!!"*
2. *„Ich bin stinksauer — zum dritten Mal dasselbe Problem und keine Lösung in Sicht. Was soll das?!"*
3. *„Euer Support ist eine absolute Katastrophe. Ich warte seit einer Woche auf eine Antwort!!!"*
