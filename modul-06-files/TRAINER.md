# Modul 06 – Files – Erklärungen

Was der Trainer am Beamer erklärt. Die Aufgaben stehen in der [README.md](README.md).

---

## Die eine Kernaussage

„KI-Nodes brauchen Text oder JSON. Die Welt liefert aber PDFs, Bilder und Audio. Dieses Modul zeigt, wie ihr die Lücke schließt.“

Die Preistabellen unten sind Nachschlagematerial, nicht zum Vorlesen. Am Beamer reichen die Entscheidungsmatrix und die Playground-Nodes.

---

## Gemini API – Preisübersicht (Paid Tier · Standard)

### Text / Multimodal
> Preise pro 1M Tokens · aufsteigend nach Input-Preis

| Modell | Model-ID | Input | Output | Kontext | n8n |
|--------|----------|-------|--------|---------|-----|
| Gemini 2.5 Flash-Lite | `gemini-2.5-flash-lite` | $0.10 ($0.30 Audio) | $0.40 | $0.01 + $1.00/h | ✅ |
| Gemini 2.5 Flash | `gemini-2.5-flash` | $0.30 ($1.00 Audio) | $2.50 | $0.03 + $1.00/h | ✅ |
| Gemini 3 Flash Preview | `gemini-3-flash-preview` | $0.50 ($1.00 Audio) | $3.00 | $0.05 + $1.00/h | ✅ |
| Gemini 2.5 Pro | `gemini-2.5-pro` | $1.25 / $2.50 >200k | $10.00 / $15.00 >200k | $0.125 + $4.50/h | ✅ |
| Gemini 3.1 Pro Preview | `gemini-3.1-pro-preview` | $2.00 / $4.00 >200k | $12.00 / $18.00 >200k | $0.20 + $4.50/h | ✅ |
| Gemini 2.0 Flash ⚠️ | `gemini-2.0-flash` | $0.10 ($0.70 Audio) | $0.40 | $0.025 + $1.00/h | ✅ |

> ⚠️ Gemini 2.0 Flash ist deprecated und wird am 01.06.2026 abgeschaltet.

---

### Bildgenerierung
> Preis pro generiertem Bild · aufsteigend

| Modell | Model-ID | Preis/Bild | Hinweis | n8n |
|--------|----------|------------|---------|-----|
| Imagen 4 Fast | `imagen-4.0-fast-generate-001` | $0.020 | Schnellste Option | ✅ |
| Gemini 2.5 Flash Image | `gemini-2.5-flash-image` | $0.039 | bis 1024×1024px | ✅ |
| Imagen 4 Standard | `imagen-4.0-generate-001` | $0.040 | – | ✅ |
| Gemini 3.1 Flash Image | `gemini-3.1-flash-image-preview` | $0.045 – $0.151 | 512px–4K skalierbar | ✅ |
| Imagen 4 Ultra | `imagen-4.0-ultra-generate-001` | $0.060 | Höchste Qualität | ✅ |
| Gemini 3 Pro Image | `gemini-3-pro-image-preview` | $0.134 – $0.240 | 1K/2K/4K | ✅ |

---

### Audio / TTS
> Preise pro 1M Tokens · 25 Tokens = 1 Sek. Audio Output

| Modell | Model-ID | Input (Text) | Output (Audio) | Typ | n8n |
|--------|----------|-------------|----------------|-----|-----|
| Gemini 2.5 Flash TTS | `gemini-2.5-flash-preview-tts` | $0.50 | $10.00 | Text-to-Speech | ✅ |
| Gemini 2.5 Flash Live | `gemini-2.5-flash-native-audio-preview` | $0.50 Text / $3.00 Audio | $2.00 Text / $12.00 Audio | Live API Echtzeit | ❌ |
| Gemini 3.1 Flash TTS | `gemini-3.1-flash-tts-preview` | $1.00 | $20.00 | Text-to-Speech | ✅ |
| Gemini 2.5 Pro TTS | `gemini-2.5-pro-preview-tts` | $1.00 | $20.00 | Text-to-Speech Pro | ✅ |
| Gemini 3.1 Flash Live | `gemini-3.1-flash-live-preview` | $0.75 Text / $3.00 Audio | $4.50 Text / $12.00 Audio | Live API Echtzeit | ❌ |

---

### Videogenerierung
> Preis pro Sekunde generiertes Video · aufsteigend (720p Basis)

| Modell | Model-ID | Preis/Sek. | Auflösungen | n8n |
|--------|----------|------------|-------------|-----|
| Veo 3.1 Lite | `veo-3.1-lite-generate-preview` | $0.05 | 720p, 1080p | ❌ |
| Veo 3.1 Fast | `veo-3.1-fast-generate-preview` | $0.10 / $0.12 / $0.30 | 720p, 1080p, 4K | ✅ |
| Veo 3 Fast | `veo-3.0-fast-generate-001` | $0.10 / $0.12 / $0.30 | 720p, 1080p, 4K | ✅ |
| Veo 2 | `veo-2.0-generate-001` | $0.35 | – | ✅ |
| Veo 3 Standard | `veo-3.0-generate-001` | $0.40 | mit Audio | ✅ |
| Veo 3.1 Standard | `veo-3.1-generate-preview` | $0.40 / $0.60 (4K) | 720p, 1080p, 4K | ✅ |

> Abrechnung nur bei erfolgreich generiertem Video. Alle Veo-Modelle nur im Paid Tier.

---

### Embeddings
> Preis pro 1M Input-Tokens

| Modell | Model-ID | Text | Weitere Modalitäten | n8n |
|--------|----------|------|---------------------|-----|
| Gemini Embedding | `gemini-embedding-001` | $0.15 | – (nur Text) | ❌ |
| Gemini Embedding 2 | `gemini-embedding-2` | $0.20 | $0.45 Bild · $6.50 Audio · $12.00 Video | ❌ |

> Batch-API: 50 % günstiger auf alle Preise. Datennutzung für Google-Training nur im Free Tier.

---

# Welche Node für welchen Input?

## Entscheidungsmatrix

| Input | Inhalt | Node | Kosten pro Seite | Warum |
|---|---|---|---|---|
| PDF (digital) | Text, Tabellen | **Extract from File** | ~$0.00 | Text direkt drin, kein API-Call |
| PDF (Scan/Foto) | Bild von Dokument | **Bild analysieren** | ~$0.004 | Braucht Vision (GPT-4o) |
| JPG/PNG | Beleg, Dokument, Szene | **Bild analysieren** | ~$0.004 | Nur Vision kann Bilder lesen |
| Audio | Gespräch, Meeting | **Transcribe Recording** | ~$0.006/min | Whisper: $0.006 pro Minute |
| Text (egal woher) | Unstrukturiert | **Information Extractor** | ~$0.001 | Extra LLM-Call auf Text |

> Preise basierend auf GPT-4o ($2.50/1M Input-Tokens) und Whisper ($0.006/min). Stand 2025.

---

## Information Extractor ist nie der Eingang — immer der Ausgang

```
PDF digital   →  Extract from File    →  Information Extractor  →  JSON
PDF Scan      →  Bild analysieren        →  Information Extractor  →  JSON
JPG           →  Bild analysieren        →  Information Extractor  →  JSON
Audio         →  Transcribe Recording →  Information Extractor  →  JSON
```

---

## Kosten sparen: Information Extractor weglassen

```
PDF digital  →  Extract from File  →  Code (Regex)                    →  JSON  ✅ ~$0.00
PDF digital  →  Extract from File  →  Basic LLM Chain (GPT-4o-mini)   →  JSON  💰 ~$0.001
JPG          →  Bild analysieren      →  (JSON direkt im Prompt)         →  JSON  💰 ~$0.004  (ein Call)
JPG          →  Bild analysieren      →  Information Extractor           →  JSON  💸 ~$0.005  (zwei Calls)
```

---

## Kosten bei 1.000 Dokumenten/Tag

| Pipeline | Kosten/Tag | Kosten/Monat |
|---|---|---|
| PDF digital → Extract from File → Code | ~$0.00 | ~$0.00 |
| PDF digital → Extract from File → Basic LLM Chain | ~$1.00 | ~$30 |
| JPG → Bild analysieren (JSON im Prompt) | ~$4.00 | ~$120 |
| JPG → Bild analysieren + Information Extractor | ~$5.00 | ~$150 |
| PDF Scan → Bild analysieren + Information Extractor | ~$5.00 | ~$150 |

---

## Faustregel

- Text im PDF markierbar? → **Extract from File** (~$0.00)
- Foto oder Scan? → **Bild analysieren** (~$0.004/Bild)
- Audio? → **Transcribe Recording** (~$0.006/min)
- Danach strukturiertes JSON ohne Prompt-Aufwand? → **Information Extractor** dranhängen (+~$0.001)

---

## Files – Dateien in Text/JSON umwandeln
- **HTTP Request:** Dateien (PDF, Bild, Audio) von einer URL als Binary laden — *Response Format: File*
- **Extract from File:** Text aus PDF, Daten aus Excel/CSV, Metadaten aus Bild extrahieren
- **Basic LLM Chain mit Bild:** Bild als Chat Message vom Typ *Image (Binary)* an ein Vision-Modell schicken, hier Azure `gpt-5-mini`
- **Transcribe Recording:** Audio per OpenAI Whisper transkribieren *(benötigt OpenAI-Credentials)*
- **Markdown:** HTML → sauberes Markdown (Token-Reduktion) oder Markdown → HTML
- **Convert to File:** JSON/Text als downloadbare Datei (.txt, .csv, .json) ausgeben
- **Code:** Token-Budget-Trimmer — Text kürzen, Tokens schätzen, Whitespace bereinigen

---

## Node für Node: zeigen und Fallen

### HTTP Request – Datei laden (`HTTP Request – PDF laden`, `HTTP Request – Bild laden`)

Einziger Unterschied zum normalen HTTP Request: **Response Format: File**. Zeigen: Node ausführen, im Output auf den Reiter **Binary** wechseln.

- Ohne *Response Format: File* kommt kein Binary an, oft ohne Fehlermeldung.
- Stimmt der Dateityp (MIME-Type) nicht, erkennt der nächste Node die Datei nicht.

### Extract from File – PDF zu Text

Kein Modell, kein API-Aufruf, keine Kosten. Der Text steht danach in `$json.text`. Zeigen: Pipeline ausführen, `text` im Output.

- Nur für **digitale** PDFs. Ein Scan ist ein Bild: `text` bleibt leer, ohne Fehler. Deshalb prüft die Challenge die Textlänge.
- Tabellen in PDFs werden oft zu Textsalat, Spalten laufen ineinander.

### Bild analysieren – Bild zu Text oder JSON

Basic LLM Chain, die das Bild als Nachricht vom Typ *Image (Binary)* an ein Vision-Modell schickt. Zwei Varianten im Playground vergleichen:

| Node | Was rauskommt | Aufrufe |
|---|---|---|
| `Bild analysieren – für Information Extractor` | freie Beschreibung, danach Information Extractor | 2 |
| `Bild analysieren – JSON im Prompt` | direkt JSON | 1, also günstiger |

- „Antworte NUR mit JSON“ halten Modelle nicht immer ein (Backticks, Einleitungssatz). Deshalb der Code-Node zum Bereinigen (`AUFGABE 08d – Code: JSON parsen`).
- Zu klein, zu dunkel, zu unscharf: schlechte Erkennung.

### Transcribe a recording – Audio zu Text

OpenAI Whisper, ca. $0.006 pro Minute. Zeigen: Audiodatei laden, Transkript im Output.

- Whisper nimmt höchstens 25 MB pro Datei. Lange Aufnahmen vorher teilen.
- Wer was gesagt hat, steht nicht im Transkript. Dafür braucht es einen eigenen Schritt mit Prompt.

### Information Extractor – Text zu JSON

Holt per Modell die Felder eines Schemas aus beliebigem Text. Das Schema steht im Node, den Prompt baut der Node selbst. Er kommt immer **nach** einem der Nodes oben, nie als Eingang.

- Fehlt ein Feld im Dokument, kommt `null`. Das ist richtig so, kein Fehler.
- Je mehr Felder, desto teurer und ungenauer. Schema schlank halten.
- Wird das Bild ohnehin analysiert: JSON gleich im Bild-Prompt anfordern, spart den zweiten Aufruf.

### Code – Token-Trimmer (`Code – Token-Trimmer`, `Code – kein Trim`, `Code – mit Trim`)

PDF-Text enthält viele Leerzeichen und Leerzeilen. Die kosten Tokens und bringen nichts. Zeigen: den `Merge` mit „kein Trim“ und „mit Trim“ nebeneinander, Zeichen, Tokens und Kosten vergleichen. Oft 30 bis 50 % weniger.

```js
raw.replace(/[ \t]+/g, ' ')    // mehrfache Leerzeichen
   .replace(/\n{3,}/g, '\n\n') // mehrfache Leerzeilen
   .trim()
```

- Zu viel kürzen zerstört Struktur, z. B. Tabellen.
- Die Token-Zahl im Code ist eine Schätzung (Wörter × 1,3).

### Übergang zur Challenge

„Ihr habt die Eingangsstrecke gesehen: Datei → Text → JSON. In der Challenge verarbeitet ihr echte Rechnungen aus einem Drive-Ordner bis ins Sheet. Ich starte die Zeit, wenn alle bereit sind.“

---

## 🧠 Learnings — Warum ist der Workflow so gebaut?

### Warum Schedule Trigger statt Google Drive Trigger?
Der Google Drive Trigger liefert pro Execution immer nur eine Datei — selbst wenn mehrere gleichzeitig hochgeladen wurden. Schedule Trigger + List Files pollt aktiv den gesamten Ordner und liefert alle Dateien auf einmal.

### Warum Loop Over Items mit Batch Size 1?
Ohne Loop laufen alle Dateien parallel. Das führt zu Race Conditions in Google Sheets — mehrere Nodes hängen gleichzeitig eine Zeile an, Einträge überschreiben sich. Batch Size 1 = eine Datei komplett durch, dann die nächste.

### Warum zwei IF-Nodes?
Zwei verschiedene Fragen:
- **IF 1 (fileExtension = pdf):** Ist es eine PDF oder ein Bild? JPGs können nicht durch Extract from File.
- **IF 2 (text.length >= 50):** Ist die PDF digital oder gedruckt/gescannt? Nur nach dem Öffnen weiß man das.

### Warum keepSource: both im Extract from File?
Standardmäßig ersetzt Extract from File das Binary durch `$json.text`. Mit `keepSource: both` bleibt das Binary erhalten — wichtig damit Bild analysieren im Bild-Zweig noch auf `$binary.data` zugreifen kann.

### Warum Stop and Error für Print-PDFs statt Bild analysieren?
Print-PDFs als Bild an Bild analysieren zu schicken funktioniert technisch, ist aber teuer und unzuverlässig — PDFs sind mehrseitig, Bild analysieren sieht nur Seite 1. Die sauberere Lösung: explizite Fehlermeldung + `Continue Regular Output` damit der Loop nicht abbricht sondern mit der nächsten Datei weitermacht.

### Warum Edit Fields (Set-Node) vor Google Sheets?
Information Extractor gibt `$json.output.rechnungsnummer` aus. Bild analysieren + Parse JSON gibt `$json.rechnungsnummer` aus. Der Set-Node mit `?.` und `??` normalisiert beide auf denselben Pfad — danach überall nur noch `$json.rechnungsnummer`.

### Warum GPT-4o-mini statt GPT-4o?
Rechnungen haben klare Struktur. GPT-4o-mini reicht aus und ist ~17x günstiger. GPT-4o lohnt sich nur bei sehr schlechter Bildqualität oder Handschrift.

### Warum Basic LLM Chain statt AI Agent?
Für einen einzelnen LLM-Call braucht man keinen Agenten. Basic LLM Chain ist einfacher, schneller, günstiger.

### Warum Move to Processed am Ende?
Verhindert Doppelverarbeitung ohne Tracking-Logik — beim nächsten Lauf listet List Files nur Dateien die noch im Eingangsordner liegen.
