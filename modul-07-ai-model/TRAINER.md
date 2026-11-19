# Modul 07 – AI + Model – Erklärungen

Was der Trainer am Beamer erklärt. Die Aufgaben stehen in der [README.md](README.md).

---

## Module 07 -  AI + Model - Node Übersicht

### KI-Modelle und Verarbeitung
- **Basic LLM Chain**: Ein einzelner Aufruf an ein Sprachmodell mit festem Prompt. Keine Tools, kein Gedächtnis.
- **Sentiment Analysis**: Analysiert die Stimmung in Texten.
- **TextClassifier**: Kategorisiert Texte nach vordefinierten Klassen.
- **InformationExtractor**: Extrahiert gezielt Informationen aus Texten.

### Sprachmodell
- **Azure AI Foundry Chat Model** (gpt-5-mini): Eingesetztes Sprachmodell zur Analyse und Verarbeitung von Texteingaben.

### Interaktion per Chat
- **When Chat Message Received**: Ermöglicht die Eingabe und Verarbeitung von Nachrichten im Chatfenster.

### Datenbankanbindung
- **Tables**: Für das Beispiel gibt es eine rudimentäre Datenhaltung in Form von n8n Tables.

---

## Einstieg

„Bis jetzt haben wir Daten umgebaut und verteilt. Jetzt lassen wir ein Sprachmodell mitdenken, aber kontrolliert. Das Ziel ist nicht Magie, sondern Vorhersehbarkeit.“

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

**Bild:** Die Basic LLM Chain ist ein Formular, das ihr ausfüllt und abschickt. Ihr wisst, was zurückkommt. Der AI Agent ist eine neue Kollegin, der ihr ein Ziel gebt. Sie entscheidet selbst, welche Werkzeuge sie dafür nimmt.

Hier fünf Minuten investieren. Sonst bauen alle reflexartig einen Agenten, auch wo eine Chain reicht.

**Frage:** Ihr wollt eine Rechnung zusammenfassen und die wichtigsten Felder herausziehen. Welcher Node?
**Antwort:** Basic LLM Chain. Die Aufgabe ist klar, das Ergebnis vorhersehbar, Tools braucht es nicht. Ein Agent bräuchte mehr Tokens und mehr Zeit und wäre weniger berechenbar.

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
(GPT, Gemini, Claude usw.) – nicht bei reinen Completion Models.

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

**Frage:** Ihr wollt dem Modell beibringen, wie es Rechnungen formatieren soll. Wie geht ihr vor?
**Antwort:** Few-Shot mit Chat Messages: eine User-Nachricht mit Beispielrechnung, eine AI-Nachricht mit genau dem gewünschten Format. Die echte Rechnung kommt ins `text`-Feld. Das Modell lernt aus dem Beispiel, nicht aus abstrakten Regeln.

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

## AI Agent

**Was er tut:** Gibt dem Modell Tools, Gedächtnis und die Freiheit, selbst die Schritte zu wählen. Er plant, ruft Tools auf, wertet aus und wiederholt das, bis er eine Antwort hat.

**Zeigen:** Agent mit einem Tool verbinden (im Workflow `GetCustomer`), System Message setzen, **Return Intermediate Steps** einschalten, ausführen: Die Tool-Aufrufe stehen im Output.

### Agent und Tools

Dem Agenten muss man nicht sagen, *dass* er ein Tool hat. n8n meldet verbundene Tools selbst an das Modell. Das Modell kennt das Tool aber nur über **Name und Beschreibung**. Eine schwache Beschreibung heißt: Das Tool wird falsch, zur falschen Zeit oder gar nicht benutzt.

```
Tool-Name:         get_invoice_data
Tool-Beschreibung: Ruft alle Rechnungen eines Kunden ab.
                   Input: customer_id (string).
                   Output: Liste mit invoice_id, amount, due_date, status.
                   Nutze dieses Tool, wenn nach offenen Rechnungen gefragt wird.
```

Die Beschreibung beantwortet: Was tut das Tool? Was braucht es? Was liefert es? Wann soll es benutzt werden?

In die **System Message** gehört: wann welches Tool dran ist (und wann nicht), was bei einem Tool-Fehler passiert (z. B. „antworte mit TOOL_ERROR“, siehe Modul 5), das Format der Antwort.

💡 **System Message = Strategie. Tool-Beschreibung = Bedienungsanleitung.** Beides muss stimmen.

**Frage:** Der Agent hat ein Google-Sheets-Tool, benutzt es aber nie, obwohl die Daten dort liegen. Woran liegt es wahrscheinlich?
**Antwort:** Die Tool-Beschreibung sagt nicht klar, wann das Tool gebraucht wird, oder die System Message erwähnt es nicht. Beschreibung schärfen und in der System Message sagen, wann das Tool zu nutzen ist.

**Fallen:**
- **Max Iterations** zu niedrig: Der Agent bricht ab, bevor er fertig ist. Bei vielen Tool-Aufrufen erhöhen.
- Der Agent ruft ein Tool immer wieder auf: Beschreibung und System Message widersprechen sich, oder er sucht etwas, das das Tool nicht liefern kann.
- Ohne *Return Intermediate Steps* seht ihr nur das Endergebnis. Beim Bauen immer einschalten.

### Options im Überblick

| Option | Wofür |
|---|---|
| **System Message** | Rolle, Regeln, Strategie |
| **Max Iterations** | Schutz vor Endlosschleifen: wie viele Tool-Runden erlaubt sind |
| **Return Intermediate Steps** | Tool-Aufrufe im Output sichtbar machen, unverzichtbar beim Debuggen |
| **Require Specific Output Format** | Zwingt den Agenten zu strukturiertem JSON |
| **Enable Fallback Model** | Zweites Modell, wenn das erste ausfällt (Modul 5) |
| **Automatically Passthrough Binary Images** | Bilder aus vorherigen Nodes gehen automatisch ans Modell |
| **Enable Streaming** | Antwort erscheint Stück für Stück, für Chat-Oberflächen |
| **Batch Processing** | Mehrere Items parallel statt nacheinander: schneller, aber mehr gleichzeitige API-Aufrufe |
| **Tracing Metadata** | Tags für Observability-Werkzeuge wie LangSmith oder Langfuse |

---

## Sentiment Analysis

Sagt, ob ein Text positiv, negativ oder neutral ist, mit Score. Text rein, Stimmung raus.

**Bild:** Hundert Support-Tickets, und ihr müsst entscheiden, welche zuerst drankommen. Sentiment Analysis sortiert vor, ohne dass jemand jedes Ticket liest.

**Zeigen:** Drei Texte eingeben, klar positiv, negativ, neutral. Scores vergleichen.

**Frage:** Wie setzt ihr das im Support ein?
**Antwort:** Als Vorstufe: stark negativ → sofort an einen Menschen. Positiv → automatische Antwort reicht. So verschwinden verärgerte Kunden nicht in der Warteschlange.

**Fallen:**
- Ironie: „Super, schon wieder ein Fehler!“ kommt oft als positiv raus. Für kritische Abläufe nur auf eindeutige Scores reagieren.
- Sehr kurze Texte („OK“, „Nein“) geben unsichere Scores.

## Text Classifier

Ordnet Text vorher festgelegten Kategorien zu. Ihr beschreibt die Kategorien, das Modell entscheidet. Jede Kategorie bekommt einen eigenen Ausgang.

**Bild:** Die Poststelle, die jeden Brief gleich ins richtige Fach legt: Beschwerde, Anfrage, Lob.

**Zeigen:** Kategorien mit Beschreibung anlegen, Text eingeben, auf die Ausgänge zeigen.

**Frage:** Was passiert mit einer Nachricht, die Beschwerde *und* Anfrage ist?
**Antwort:** Standardmäßig gewinnt die wahrscheinlichste Kategorie. Sollen mehrere Ausgänge feuern, muss man das in den Optionen erlauben.

**Fallen:**
- Zu ähnliche Kategorien („Beschwerde“ und „Kritik“): Das Modell rät.
- Zu knappe Beschreibungen. Besser: „Beschwerde: Kunde meldet einen Fehler oder ist mit dem Service unzufrieden und erwartet eine Lösung.“
- Ohne Ausweichkategorie („Sonstiges“) fallen Texte, die nirgends passen, durch.

## Information Extractor

Holt strukturierte Felder aus freiem Text, nach einem Schema im Node.

**Bild:** „Hallo, ich bin Max Mustermann, Rechnung 4711 stimmt nicht.“ wird zu `{ "name": "Max Mustermann", "rechnungsnummer": "4711" }`, ohne Regex.

**Zeigen:** Schema mit `name` und `Rechnungsnummer`, Chat-Eingabe als Text, Ergebnis neben den Rohtext legen.

**Frage:** Was passiert, wenn der Name im Text fehlt?
**Antwort:** Der Node bricht nicht ab, das Feld bleibt leer bzw. `null`. Danach mit einem IF prüfen und z. B. nachfragen.

**Fallen:**
- Jedes zusätzliche Feld kostet Tokens und macht die übrigen ungenauer. Nur holen, was weiterverarbeitet wird.
- Feldnamen in `snake_case` oder `camelCase`, keine Leer- oder Sonderzeichen wie in `Betrag (brutto)`.
- Er braucht Text als Eingang, kommt also immer nach Chat, Extract from File, Bild analysieren oder einem anderen Modell.

---

## Übergang zur Challenge

„Ihr kennt jetzt die Bausteine, vom einfachen Modellaufruf bis zum Agenten mit Tools. In der Challenge baut ihr einen Support-Workflow: Eingabe per Chat, Stimmung und Kategorie erkennen, passend antworten. Ich starte die Zeit, wenn alle bereit sind.“

---

## Weiterführende Links

- [n8n Docs – Basic LLM Chain](https://docs.n8n.io/integrations/builtin/cluster-nodes/root-nodes/n8n-nodes-langchain.chainllm/)
- [n8n Docs – Agents vs. Chains](https://docs.n8n.io/advanced-ai/examples/agent-chain-comparison/)
- [n8n Docs – Was ist eine Chain?](https://docs.n8n.io/advanced-ai/examples/understand-chains/)
- [DEV.to – n8n AI Agent vs LLM Chain](https://dev.to/ciphernutz/n8n-ai-agent-vs-llm-chain-when-to-use-langchain-code-48h4)
- [logicworkflow.com – Basic LLM Chain Best Practices](https://logicworkflow.com/nodes/basic-llm-chain-node/)
