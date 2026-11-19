# Modul 07 – AI + Model

Dieses Handout ist der Text der Sticky Notes aus dem Workflow, damit ihr ihn im zweiten Browser-Tab lesen und Code kopieren könnt.

**Workflow-Datei:** [`modul-07-ai-model.json`](modul-07-ai-model.json)

---

## Module 07 -  AI + Model - Node Übersicht

### KI-Modelle und Verarbeitung
- **LLM Chain**: Verknüpft mehrere Sprachmodelle für komplexe Aufgaben.
- **Sentiment Analysis**: Analysiert die Stimmung in Texten.
- **TextClassifier**: Kategorisiert Texte nach vordefinierten Klassen.
- **InformationExtractor**: Extrahiert gezielt Informationen aus Texten.

### Sprachmodell
- **Google Gemini Chat Model**: Eingesetztes Sprachmodell zur Analyse und Verarbeitung von Texteingaben.

### Interaktion per Chat
- **When Chat Message Received**: Ermöglicht die Eingabe und Verarbeitung von Nachrichten im Chatfenster.

### Datenbankanbindung
- **Tables*: Für das Beispiel gibt es eine rudimentäre Datenhaltung in Form von n8n Tables.

---

# n8n AI Nodes – Best Practices

---

## Basic LLM Chain vs. AI Agent

| | Basic LLM Chain | AI Agent |
|---|---|---|
| **Wofür** | Einmaliger, definierter Task | Mehrere Schritte, Entscheidungen, Tools |
| **Memory** | Nein (stateless) | Ja – über Memory-Node |
| **Tools** | Nein | Ja – Wikipedia, HTTP, Sheets… |
| **Chat Messages** | Ja – manuell aufbaubar | Nein – Agent verwaltet intern |
| **Determinismus** | Vorhersehbar | Nicht-deterministisch |
| **Token-Kosten** | Niedrig | Höher (mehrere Runden) |

### Faustregel

```
Fester Ablauf + feste Struktur   →  Basic LLM Chain
Eigene History manuell aufbauen  →  Basic LLM Chain + Chat Messages
Agent entscheidet selbst         →  AI Agent + Memory Node
```

---

## Source for Prompt (User Message)

Steuert **woher** der User-Prompt kommt. Drei Optionen:

### Define below
Du schreibst den Prompt selbst – statisch oder per Expression.

```
Analysiere diese Rechnung: {{ $json.invoiceText }}
```

**Wann nutzen:** Immer wenn du den Prompt kontrollieren willst –
also in 90% aller Fälle.

### Connected Chat Trigger Node
n8n liest `chatInput` automatisch aus dem verbundenen Chat Trigger.
Du musst nichts referenzieren – kein `$('When chat message received').item.json.chatInput`.

**Wann nutzen:** Wenn dein Workflow direkt mit einem
`When chat message received`-Trigger startet und der User-Text
ungefiltert als Prompt dienen soll.

> ⚠️ Funktioniert nur wenn das eingehende Feld exakt `chatInput` heißt.
> Sonst vorher einen Set-Node schalten und das Feld umbenennen.

### Take from previous node automatically
n8n erwartet ebenfalls ein Feld namens `chatInput` vom vorherigen Node.

**Wann nutzen:** Wenn ein anderer Node (kein Chat Trigger)
einen fertigen Prompt-Text produziert und dieser direkt übergeben werden soll.

### Entscheidungsbaum

```
Startet der Workflow mit einem Chat Trigger?
  → Ja, User-Text soll direkt als Prompt →  Connected Chat Trigger Node
  → Ja, aber du willst Kontext ergänzen   →  Define below + {{ $json.chatInput }}
  → Nein, Daten kommen aus anderem Node   →  Define below oder Take from previous node
```

---

## Chat Messages (if Using a Chat Model)

Erscheint **nur wenn ein Chat Model** angeschlossen ist
(GPT-4, Gemini, Claude usw.) – nicht bei reinen Completion Models.

Chat Models erwarten keine einzelne Texteingabe,
sondern ein **Array von Nachrichten mit Rollen**.
Genau das baut dieser Abschnitt manuell auf.

> n8n ignoriert diese Einstellungen komplett,
> wenn kein Chat Model verbunden ist.

### Die drei Message-Typen

#### System
Setzt Persona, Regeln und Verhalten des Modells.
Gilt für die gesamte Konversation und kann vom User nicht überschrieben werden.

```
You are a financial support analyst.
Answer only based on the invoice data provided.
Be precise and concise. Never invent data.
```

**Wo konfigurieren:** Nicht in Chat Messages, sondern unter
`Options → System Message` – einfacher, macht dasselbe.
Chat Messages → System nur wenn du die Message dynamisch
per Expression befüllen willst.

#### User (HumanMessagePromptTemplate)
Repräsentiert eine Nutzernachricht in der simulierten History.
Wird für Few-Shot-Beispiele genutzt – zeigt dem Modell wie
eine Frage aussieht.

#### AI (AIMessagePromptTemplate)
Repräsentiert eine **simulierte Assistenten-Antwort** in der History.
Wird zusammen mit User-Messages für Few-Shot-Prompting verwendet –
zeigt dem Modell wie die Antwort aussehen soll.

> ⚠️ AI Message ≠ Anweisung. Sie simuliert eine bereits gegebene
> Antwort. Als Regel oder Instruction ist sie falsch platziert.

### Few-Shot Pattern (der einzige sinnvolle Anwendungsfall)

```
Options → System Message  →  Rolle + Regeln (statisch)
Chat Messages:
  User  →  Beispielfrage
  AI    →  Beispielantwort (genau so soll das Modell antworten)
  User  →  Beispielfrage 2
  AI    →  Beispielantwort 2

text-Feld (Prompt):  →  Der eigentliche Task mit {{ $json.data }}
```

### Wann Chat Messages weglassen?

In den meisten Fällen. Wenn du nur:
- eine System Message brauchst → `Options → System Message`
- einen User-Prompt brauchst → `text`-Feld (Source for Prompt)

Dann sind Chat Messages **nicht nötig**.
Chat Messages nur nutzen wenn du dem Modell
konkrete Beispiele (Few-Shot) mitgeben willst.

### Häufige Fehler

| Fehler | Problem | Richtig |
|---|---|---|
| System-Prompt als erste Chat Message ohne type | Wird als User-Nachricht interpretiert | `Options → System Message` nutzen |
| AI Message als Anweisung benutzen | AI Message = simulierte Antwort, keine Regel | Anweisung in System Message oder text-Feld |
| Chat Messages + kein Chat Model | Wird komplett ignoriert | Completion Model → Chat Messages weglassen |
| Daten im text-Feld vergessen einzubetten | Modell antwortet ohne Kontext | Expression `{{ $json.field }}` einfügen |

---

## Vollständiges Konfigurationsschema (Basic LLM Chain)

```
Source for Prompt
  └── Define below
        └── text: "Rechnungsdaten:\n{{ $json.invoiceData }}\n\nFrage: {{ $json.chatInput }}"

Options
  └── System Message: "You are a financial support analyst. ..."

Chat Messages  ← nur bei Few-Shot nötig
  └── User:  "Welche Rechnungen sind offen für Kunde XY?"
  └── AI:    "Kunde XY hat zwei offene Rechnungen: ..."
```

---

## AI Agent: Options im Überblick

| Option | Wofür |
|---|---|
| **System Message** | Persona/Rolle des Agents |
| **Max Iterations** | Schutz vor Endlosschleifen (Tool-Call-Limit) |
| **Return Intermediate Steps** | Zeigt Tool-Calls im Output – wichtig zum Debuggen |
| **Require Specific Output Format** | Zwingt Agent zu strukturiertem JSON |
| **Enable Fallback Model** | Zweites Modell wenn erstes fehlschlägt |

---

## Weiterführende Links

- [n8n Docs – Basic LLM Chain](https://docs.n8n.io/integrations/builtin/cluster-nodes/root-nodes/n8n-nodes-langchain.chainllm/)
- [n8n Docs – Agents vs. Chains](https://docs.n8n.io/advanced-ai/examples/agent-chain-comparison/)
- [n8n Docs – Was ist eine Chain?](https://docs.n8n.io/advanced-ai/examples/understand-chains/)
- [DEV.to – n8n AI Agent vs LLM Chain](https://dev.to/ciphernutz/n8n-ai-agent-vs-llm-chain-when-to-use-langchain-code-48h4)
- [logicworkflow.com – Basic LLM Chain Best Practices](https://logicworkflow.com/nodes/basic-llm-chain-node/)

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

