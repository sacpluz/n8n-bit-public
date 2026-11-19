# Modul 07 – AI + Model – Erklärungen

Was der Trainer am Beamer erklärt. Die Aufgaben stehen in der [README.md](README.md).

---

## Module 07 -  AI + Model - Node Übersicht

### KI-Modelle und Verarbeitung
- **LLM Chain**: Verknüpft mehrere Sprachmodelle für komplexe Aufgaben.
- **Sentiment Analysis**: Analysiert die Stimmung in Texten.
- **TextClassifier**: Kategorisiert Texte nach vordefinierten Klassen.
- **InformationExtractor**: Extrahiert gezielt Informationen aus Texten.

### Sprachmodell
- **Azure AI Foundry Chat Model** (gpt-5-mini): Eingesetztes Sprachmodell zur Analyse und Verarbeitung von Texteingaben.

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
