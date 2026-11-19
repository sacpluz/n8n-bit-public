# Modul 05 – Error Handling – Erklärungen

Was der Trainer am Beamer erklärt. Die Aufgaben stehen in der [README.md](README.md).

---

## Einstieg

„Bis jetzt haben wir Workflows gebaut, die funktionieren. Jetzt schauen wir, was passiert, wenn sie es nicht tun, und wie n8n damit umgeht, statt einfach abzustürzen.“

Das Modul ist keine Pipeline, sondern mehrere unabhängige Demo-Blöcke auf dem Canvas. Der Reihe nach:

| Schritt | Block | Nodes im Workflow |
|---|---|---|
| 1 | **On Error**: Stop, Continue, Error Output | `Edit Fields – Fehler auslösen (Stop)` mit Folgenode, dieselben Nodes mit Endung `1` und `2` |
| 2 | **Retry on Fail** | `HTTP Request – Retry on Fail - Funktioniert`, `HTTP Request – Retry on Fail1` |
| 3 | **Weitere Settings** | Always Output Data, Execute Once (an einem beliebigen Node zeigen) |
| 4 | **Stop and Error** | `Edit Fields – Kundendaten setzen` → `IF – Ist Kunde?` → `Stop and Error – Not allowed` |
| 5 | **Error Trigger Workflow** | `Error Trigger` → `Log Error` und `Gmail – Admin benachrichtigen` |
| 6 | **AI Agent** | `AI Agent` mit `OpenAI Chat Model – Primär` und `Anthropic Chat Model – Fallback`; `AI Agent – Tool Fehlerbehandlung` mit `Gmail – Send Email Tool (Deaktiviert)` |
| 7 | **Debug Helper und Execution Log** | `DebugHelper – throwError` |

---

## 1 · Node Settings: On Error

Jeder Node hat unter **Settings → On Error** drei Möglichkeiten. Demo-Fehler: `Spotify_Streams` soll eine Zahl sein, bekommt aber den Text aus `Band`. Das ist ein Typ-Konflikt.

| Option | Was passiert | Wann richtig |
|---|---|---|
| **Stop Workflow** (Standard) | Roter Node, Workflow hält an, Folgenodes laufen nicht | Wenn mit fehlerhaften Daten nichts weiterlaufen darf: Pflichtfeld fehlt, Key ungültig |
| **Continue** (Regular Output) | Workflow läuft weiter, der Fehler steht als `error`-Objekt im Output | Wenn ein einzelnes kaputtes Item tolerierbar ist. Nie als Standard für alles |
| **Continue** (Using Error Output) | Der Node bekommt einen zweiten, roten Ausgang. Fehlerhafte Items gehen dorthin, der Rest normal weiter | Wenn Fehler vorkommen dürfen, aber behandelt werden müssen: loggen, melden, nachholen |

**Fragen an die Gruppe:**

- *Was ist das Risiko, wenn ihr überall Continue einstellt?* Ihr seht keine Fehler mehr, sie passieren aber trotzdem. Kaputte Daten landen im Sheet oder beim Kunden, und der nächste Node scheitert an fehlenden Feldern.
- *Was bringt Error Output gegenüber Continue?* Kontrolle: Der Fehler hat einen eigenen Weg, auf dem ihr ihn loggt, meldet oder erneut versucht.

**Fallen:**
- **Stop ohne Error Workflow** heißt: Der Workflow steht, und niemand merkt es.
- n8n prüft Typen streng. Auch `"42"` als Text in einem Zahlenfeld ist ein Fehler.
- Bei Stop hält der ganze Lauf an, nicht nur das fehlerhafte Item. Error Output arbeitet dagegen pro Item: Drei von fünf schlagen fehl, genau diese drei gehen in den roten Zweig.
- Bei Continue sieht der Node grün aus. Nur der Blick in den Output zeigt das `error`-Objekt.
- Error Output nicht mit **Stop and Error** verwechseln: Das eine fängt Fehler ab, das andere wirft absichtlich einen.
- Den roten Ausgang auch anschließen, sonst landen die Fehler nirgends.

## 2 · Retry on Fail

**Settings → Retry On Fail**: Anzahl Versuche und Pause dazwischen.

- `HTTP Request – Retry on Fail - Funktioniert`: gültige URL, läuft normal durch.
- `HTTP Request – Retry on Fail1`: URL gibt es nicht. Drei Versuche, dann Fehler. Während der Versuche auf das Kreis-Symbol am Node zeigen.

**Frage:** Wofür lohnt sich Retry?
**Antwort:** Für alles, was über das Netz geht und kurz ausfallen kann: HTTP Request, Gmail, Sheets, Datenbanken. Besonders bei `429 Too Many Requests`. Bei Logikfehlern (falscher Typ, fehlendes Feld, falscher Key) hilft auch der zehnte Versuch nicht.

**Fallen:**
- Pause × Versuche × Items: Bei 100 Items, 3 Versuchen und 2 Sekunden Pause dauert ein Lauf schnell Minuten.
- Retry und On Error arbeiten zusammen: On Error greift erst, wenn alle Versuche gescheitert sind.

## 3 · Weitere Settings

| Setting | Was es tut | Falle |
|---|---|---|
| **Always Output Data** | Gibt ein leeres Item aus, auch wenn nichts ankommt. So laufen Folgenodes trotzdem (z. B. Aufräumen, Meldung „nichts gefunden“) | Der Folgenode bekommt ein Item ohne Felder. Prüfen, ob er damit klarkommt |
| **Execute Once** | Der Node läuft nur für das erste Item. Gut für einmalige Schritte wie Konfiguration laden | Die übrigen Items werden ohne Hinweis ignoriert |

## 4 · Stop and Error

Wirft absichtlich einen Fehler mit eigener, lesbarer Meldung. Im Workflow: Kunde `Max Mustermann`, `is_customer` = false → `IF – Ist Kunde?` → false → `Stop and Error – Not allowed` mit der Meldung „Not allowed – Max Mustermann ist kein registrierter Kunde“. Die Meldung ist eine Expression und enthält den Namen.

**Frage:** Warum ist das besser, als am false-Zweig einfach nichts anzuschließen?
**Antwort:** Ohne Node verschwinden die Items still. Mit Stop and Error gibt es einen lesbaren Fehler im Execution Log, und der Error Workflow kann ihn auffangen. Erst dann ist Überwachung möglich.

**Fallen:**
- Feld **Error Message** auf *Expression* stellen, sonst steht `{{ … }}` wörtlich in der Meldung.
- Nach Stop and Error läuft nichts mehr. Was trotzdem passieren soll, gehört in einen Error-Output-Zweig davor.

## 5 · Error Trigger Workflow

Ein **eigener** Workflow, der startet, wenn ein anderer Workflow scheitert. Einrichten: im fehleranfälligen Workflow **⋯ → Settings → Error Workflow** diesen Workflow auswählen. Der Error Workflow selbst muss nicht aktiv sein.

Im Workflow: `Error Trigger` → `Log Error` (Google Sheets: Zeit, Workflow, Link, Node, Meldung) und `Gmail – Admin benachrichtigen`.

**Felder im Error Trigger:**

```
$json.workflow.id
$json.workflow.name
$json.execution.id
$json.execution.url                 Link zum gescheiterten Lauf
$json.execution.error.message
$json.execution.lastNodeExecuted    Node, an dem es scheiterte
```

Scheitert schon der Trigger des Haupt-Workflows, sehen die Daten anders aus (`$json.trigger.error`). Fallbacks einbauen, z. B. `{{ $json.execution.lastNodeExecuted || "unbekannt" }}`.

**Frage:** Warum ein zentraler Error Workflow statt Fehlerbehandlung in jedem Workflow?
**Antwort:** Einmal bauen, einmal pflegen. Ändert sich die Admin-Adresse, ändert ihr sie an einer Stelle statt in zwanzig Workflows. Alle Fehler landen in einem einheitlichen Log.

**Fallen:**
- Der Error Workflow startet nur bei **produktiven** Läufen (aktiver Workflow), nicht bei *Execute workflow* im Editor. Zum Testen den Haupt-Workflow kurz aktivieren oder den `Error Trigger` per *Execute step* mit Beispieldaten ausführen.
- Scheitert der Error Workflow selbst (z. B. falsche Sheet-ID), meldet niemand mehr etwas. Ihn besonders sorgfältig testen.

## 6 · AI Agent: Fehlerbehandlung

Ein Agent kann an drei Stellen scheitern: Der Modell-Anbieter fällt aus, ein Tool schlägt fehl, oder der Agent-Node selbst wirft einen Fehler.

**Fallback Model** (`AI Agent`): Unter *Enable Fallback Model* ein zweites Modell anschließen, hier OpenAI als primäres, Anthropic als Ersatz.
- Immer einen **anderen Anbieter** nehmen. Fällt OpenAI aus, sind alle OpenAI-Modelle weg.
- Prompts können sich auf dem Ersatzmodell anders verhalten. Mit beiden testen.

**Tool-Fehler** (`AI Agent – Tool Fehlerbehandlung`): Das Gmail-Tool ist deaktiviert. Die System Message legt fest: Wenn keine E-Mail gesendet wird, antworte mit „email not sent tool error“. Danach kann ein IF auf diesen Text prüfen.

**Frage:** Warum reicht On Error am Agent-Node nicht?
**Antwort:** On Error fängt nur Fehler, die den Node selbst abstürzen lassen, etwa eine abgebrochene Verbindung zum Modell. Scheitert ein Tool, läuft der Agent weiter und schreibt irgendeine Erklärung in Prosa, die kein IF zuverlässig prüfen kann. Deshalb ein fester Fehlertext in der System Message.

**Fallen:**
- Modelle halten sich nicht immer exakt an den Text („Email not sent - tool error“). Im IF mit *contains* prüfen statt *equals*, oder ein Beispiel in den Prompt schreiben.
- Ein deaktiviertes Tool und ein Tool ohne Credential sind zwei verschiedene Fehler. Beide testen.

## 7 · Debug Helper und Execution Log

**`DebugHelper – throwError`** wirft absichtlich einen Fehler. So testet ihr den Error Workflow, ohne auf einen echten Fehler zu warten.

1. Haupt-Workflow aktivieren, DebugHelper auslösen → Lauf scheitert.
2. Error Workflow startet.
3. Fehler steht im Sheet und kommt per Mail.

Danach den Debug Helper entfernen oder deaktivieren. Er wirft bei jedem Lauf einen Fehler.

**Execution Log:** Unter **Executions** lässt sich jeder vergangene Lauf öffnen. **Debug in Editor** lädt die Daten eines gescheiterten Laufs in den Editor. So lassen sich Fehler aus dem Betrieb nachstellen, die lokal schwer zu reproduzieren sind. Läufe aus dem Editor sind als Test markiert, produktive Läufe nicht.

---

## Übergang zur Challenge

„Ihr habt alle Ebenen gesehen, vom einzelnen Node bis zum zentralen Error Workflow. In der Challenge baut ihr einen Workflow, der bewusst mit Fehlern umgeht: ein Weg für den guten Fall, einer für den schlechten. Ich starte die Zeit, wenn alle bereit sind.“
