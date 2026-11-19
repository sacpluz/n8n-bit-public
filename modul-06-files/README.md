# Modul 06 – Files

Dieses Handout ist der Text der Sticky Notes aus dem Workflow, damit ihr ihn im zweiten Browser-Tab lesen und Code kopieren könnt.

**Workflow-Datei:** [`modul-06-files.json`](modul-06-files.json)

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
| PDF (Scan/Foto) | Bild von Dokument | **Analyze Image** | ~$0.004 | Braucht Vision (GPT-4o) |
| JPG/PNG | Beleg, Dokument, Szene | **Analyze Image** | ~$0.004 | Nur Vision kann Bilder lesen |
| Audio | Gespräch, Meeting | **Transcribe Recording** | ~$0.006/min | Whisper: $0.006 pro Minute |
| Text (egal woher) | Unstrukturiert | **Information Extractor** | ~$0.001 | Extra LLM-Call auf Text |

> Preise basierend auf GPT-4o ($2.50/1M Input-Tokens) und Whisper ($0.006/min). Stand 2025.

---

## Information Extractor ist nie der Eingang — immer der Ausgang

```
PDF digital   →  Extract from File    →  Information Extractor  →  JSON
PDF Scan      →  Analyze Image        →  Information Extractor  →  JSON
JPG           →  Analyze Image        →  Information Extractor  →  JSON
Audio         →  Transcribe Recording →  Information Extractor  →  JSON
```

---

## Kosten sparen: Information Extractor weglassen

```
PDF digital  →  Extract from File  →  Code (Regex)                    →  JSON  ✅ ~$0.00
PDF digital  →  Extract from File  →  Basic LLM Chain (GPT-4o-mini)   →  JSON  💰 ~$0.001
JPG          →  Analyze Image      →  (JSON direkt im Prompt)         →  JSON  💰 ~$0.004  (ein Call)
JPG          →  Analyze Image      →  Information Extractor           →  JSON  💸 ~$0.005  (zwei Calls)
```

---

## Kosten bei 1.000 Dokumenten/Tag

| Pipeline | Kosten/Tag | Kosten/Monat |
|---|---|---|
| PDF digital → Extract from File → Code | ~$0.00 | ~$0.00 |
| PDF digital → Extract from File → Basic LLM Chain | ~$1.00 | ~$30 |
| JPG → Analyze Image (JSON im Prompt) | ~$4.00 | ~$120 |
| JPG → Analyze Image + Information Extractor | ~$5.00 | ~$150 |
| PDF Scan → Analyze Image + Information Extractor | ~$5.00 | ~$150 |

---

## Faustregel

- Text im PDF markierbar? → **Extract from File** (~$0.00)
- Foto oder Scan? → **Analyze Image** (~$0.004/Bild)
- Audio? → **Transcribe Recording** (~$0.006/min)
- Danach strukturiertes JSON ohne Prompt-Aufwand? → **Information Extractor** dranhängen (+~$0.001)

---

## Files – Dateien in Text/JSON umwandeln
- **HTTP Request:** Dateien (PDF, Bild, Audio) von einer URL als Binary laden — *Response Format: File*
- **Extract from File:** Text aus PDF, Daten aus Excel/CSV, Metadaten aus Bild extrahieren
- **Analyze Image:** Bild per OpenAI Vision analysieren und beschreiben *(benötigt OpenAI-Credentials)*
- **Transcribe Recording:** Audio per OpenAI Whisper transkribieren *(benötigt OpenAI-Credentials)*
- **Markdown:** HTML → sauberes Markdown (Token-Reduktion) oder Markdown → HTML
- **Convert to File:** JSON/Text als downloadbare Datei (.txt, .csv, .json) ausgeben
- **Code:** Token-Budget-Trimmer — Text kürzen, Tokens schätzen, Whitespace bereinigen

---

# Files – Rechnungs-Automatisierung Challenge
## Automatischer Rechnungs-Parser mit Google Drive

**Dauer:** 20–25 Min | **Team:** 2 Personen | **Nodes:** 14 Nodes

---

## 🎯 Szenario

Täglich landen Rechnungen im Google Drive — als digitales PDF, als Scan-PDF oder als Bild (JPG/PNG). Baut einen vollautomatischen Workflow der alle Rechnungen verarbeitet, per KI die Daten extrahiert, in Google Sheets speichert, eine E-Mail verschickt und die Datei danach verschiebt.

**Besonderheit:** Der Workflow erkennt automatisch ob es ein digitales PDF, ein Scan oder ein Bild ist — und wählt den passenden Weg. Print-PDFs (gescannte PDFs ohne extrahierbaren Text) werden mit einer klaren Fehlermeldung gestoppt.

---

## Ordner & Test-Dateien

| Was | Wo |
|---|---|
| Eingangsordner | Google Drive → `Rechnungen n8n` |
| Zielordner | Google Drive → `Rechnungen verarbeitet` |
| Test: Digitales PDF | `wordpress-pdf-invoice-plugin-sample.pdf` |
| Test: Print-PDF | `beispielrechnung-print-01.pdf` |

---

## Die komplette Pipeline

```
Schedule Trigger
→ Google Drive List Files
→ Loop Over Items (Batch 1)
  → Google Drive Download
  → IF: PDF oder Bild? (fileExtension = pdf)
      true (PDF)  → Extract from File  ← keepSource: both!
                    → IF: Digital oder Scan? (text.length >= 50)
                        true  → Information Extractor (GPT-4o-mini)
                                → Edit Fields
                        false → Stop and Error  ← Print-PDF!
      false (Bild) → Analyze Image (base64)
                    → Parse Image JSON
                    → Edit Fields
  → Google Sheets
  → Basic LLM Chain (Gemini)
  → Gmail
  → Move to Processed
  → [zurück zu Loop]
```

---

## Aufgaben

### 01 · SCHEDULE TRIGGER — Workflow alle 5 Minuten starten

> → Field: `Minutes` · Every: `5`

---

### 02 · GOOGLE DRIVE LIST FILES — Ordnerinhalt abrufen

> → Resource: `File/Folder` · Operation: `Get Many`
> → Filter: Ordner `Rechnungen n8n`
> → Options → Fields: `*`

---

### 03 · LOOP OVER ITEMS — Nacheinander verarbeiten

> → Batch Size: `1`

⚠️ Move to Processed am Ende zurück zu **Input 0** des Loop verbinden!

---

### 04 · GOOGLE DRIVE DOWNLOAD — Datei laden

> → Operation: `Download`
> → File ID: `={{ $json.id }}`
> → Options → File Name: `={{ $json.name }}`

---

### 05 · IF: PDF ODER BILD? — Dateityp prüfen

> → Bedingung: `{{ $json.fileExtension }}` equals `pdf`
> → true → Extract from File
> → false → Analyze Image

---

### 06 · EXTRACT FROM FILE — Text extrahieren *(PDF-Zweig)*

> → Operation: `PDF`
> → **Options → Keep Source: Both** ← wichtig!

Warum? Damit das Binary erhalten bleibt falls Analyze Image es später braucht.

---

### 07 · IF: DIGITAL ODER SCAN? — Textinhalt prüfen

> → Bedingung: `{{ $json.text.trim().length }}` >= `50` (Typ: Number)
> → true → Information Extractor
> → false → Stop and Error

---

### 08a · INFORMATION EXTRACTOR — Felder aus Text *(digitale PDFs)*

Verbinde **OpenAI Chat Model GPT-4o-mini** als Subnode.

> → Text: `{{ $json.text }}`

**Felder:**

| Name | Typ | Beschreibung |
|---|---|---|
| rechnungsnummer | String | Die Rechnungsnummer |
| datum | String | Rechnungsdatum TT.MM.JJJJ |
| faelligkeitsdatum | String | Fälligkeitsdatum |
| lieferant | String | Name des Lieferanten |
| betrag_netto | Number | Nettobetrag ohne MwSt |
| mwst_satz | Number | MwSt-Satz in Prozent |
| betrag_brutto | Number | Gesamtbetrag inkl. MwSt |
| waehrung | String | Währung z.B. EUR |
| kategorie | String | z.B. Software, Reise, Bewirtung |
| quelle | String | Immer: `Digitales PDF` |

---

### 08b · STOP AND ERROR — Print-PDF abfangen *(false-Zweig)*

> → Error Message: `Das PDF ist kein digitales, sondern ein Print. Das Bild muss zu einem Bild konvertiert werden.`
> → Settings → **On Error: Continue Regular Output**

Warum `Continue Regular Output`? Damit der Loop nicht abbricht sondern mit der nächsten Datei weitermacht.

---

### 08c · ANALYZE IMAGE — Bild analysieren *(Bild-Zweig)*

> → Model: `GPT-4o-mini` · Resource: `Image` · Operation: `Analyze`
> → Input Type: `Base64`

**Prompt:**
```
Du bist ein Buchhaltungs-Assistent. Extrahiere alle relevanten Daten.

Antworte NUR mit validem JSON, keine Backticks:
{
  "rechnungsnummer": "",
  "datum": "",
  "faelligkeitsdatum": "",
  "lieferant": "",
  "betrag_netto": 0,
  "mwst_satz": 0,
  "betrag_brutto": 0,
  "waehrung": "EUR",
  "kategorie": "",
  "quelle": "Scan/Bild",
  "lesbar": true
}
Falls ein Wert fehlt: null. Falls unleserlich: lesbar: false.
```

---

### 08d · CODE: JSON PARSEN — Analyze Image Output bereinigen

```javascript
const raw = $json.text || $json.content || '';
const cleaned = raw.replace(/```json\n?/g, '').replace(/```/g, '').trim();
try {
  return [{ json: JSON.parse(cleaned) }];
} catch(e) {
  return [{ json: { parse_error: true, raw: cleaned } }];
}
```

---

### 09 · EDIT FIELDS (SET) — Felder normalisieren

Beide Wege liefern unterschiedliche Strukturen:
- Information Extractor → `$json.output.rechnungsnummer`
- Analyze Image + Parse JSON → `$json.rechnungsnummer`

**Mode: Raw JSON:**
```json
{
  "rechnungsnummer":   "{{ $json.output?.rechnungsnummer ?? $json.rechnungsnummer }}",
  "datum":             "{{ $json.output?.datum ?? $json.datum }}",
  "faelligkeitsdatum": "{{ $json.output?.faelligkeitsdatum ?? $json.faelligkeitsdatum }}",
  "lieferant":         "{{ $json.output?.lieferant ?? $json.lieferant }}",
  "betrag_netto":      "{{ $json.output?.betrag_netto ?? $json.betrag_netto }}",
  "mwst_satz":         "{{ $json.output?.mwst_satz ?? $json.mwst_satz }}",
  "betrag_brutto":     "{{ $json.output?.betrag_brutto ?? $json.betrag_brutto }}",
  "waehrung":          "{{ $json.output?.waehrung ?? $json.waehrung }}",
  "kategorie":         "{{ $json.output?.kategorie ?? $json.kategorie }}",
  "quelle":            "{{ $json.output?.quelle ?? $json.quelle }}"
}
```

---

### 10 · GOOGLE SHEETS — Daten eintragen

> → Operation: `Append` · Sheet: `Rechnungen`
> → Alle Felder: `{{ $json.rechnungsnummer }}` etc. *(kein `.output` dank Edit Fields)*
> → Dateiname: `={{ $('Google Drive – Download').item.json.name }}`
> → Verarbeitet am: `={{ new Date().toLocaleString('de-DE', {timeZone: 'Europe/Berlin'}) }}`

---

### 11 · BASIC LLM CHAIN — E-Mail-Text generieren

Verbinde **Google Gemini Chat Model** als Subnode.

**Prompt:**
```
Du bist ein Buchhaltungs-Assistent. Erstelle eine kurze professionelle
E-Mail-Zusammenfassung auf Deutsch.

- Lieferant: {{ $json.Lieferant }}
- Rechnungsnummer: {{ $json.Rechnungsnummer }}
- Datum: {{ $json.Datum }}
- Fällig am: {{ $json.Fälligkeitsdatum }}
- Betrag brutto: {{ $json['Betrag Brutto'] }} {{ $json.Währung }}
- Kategorie: {{ $json.Kategorie }}

2-3 Satz Zusammenfassung + Hinweis falls Fälligkeit < 7 Tage + Zahlungsempfehlung.
Kein JSON, nur Text.
```

---

### 12 · GMAIL — Benachrichtigung senden

> → Subject: `✅ Rechnung: {{ $('Google Sheets2').item.json.Rechnungsnummer }} - {{ $('Google Sheets2').item.json.Lieferant }}`
> → Message: `{{ $json.text }}`

---

### 13 · GOOGLE DRIVE MOVE — Datei verschieben

> → Operation: `Move`
> → File ID: `={{ $('Google Drive – List Files').item.json.id }}`
> → Folder: `Rechnungen verarbeitet`

⚠️ Danach zurück zu **Loop Over Items (Input 0)** verbinden!

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
Standardmäßig ersetzt Extract from File das Binary durch `$json.text`. Mit `keepSource: both` bleibt das Binary erhalten — wichtig damit Analyze Image im Bild-Zweig noch auf `$binary.data` zugreifen kann.

### Warum Stop and Error für Print-PDFs statt Analyze Image?
Print-PDFs als Bild an Analyze Image zu schicken funktioniert technisch, ist aber teuer und unzuverlässig — PDFs sind mehrseitig, Analyze Image sieht nur Seite 1. Die sauberere Lösung: explizite Fehlermeldung + `Continue Regular Output` damit der Loop nicht abbricht sondern mit der nächsten Datei weitermacht.

### Warum Edit Fields (Set-Node) vor Google Sheets?
Information Extractor gibt `$json.output.rechnungsnummer` aus. Analyze Image + Parse JSON gibt `$json.rechnungsnummer` aus. Der Set-Node mit `?.` und `??` normalisiert beide auf denselben Pfad — danach überall nur noch `$json.rechnungsnummer`.

### Warum GPT-4o-mini statt GPT-4o?
Rechnungen haben klare Struktur. GPT-4o-mini reicht aus und ist ~17x günstiger. GPT-4o lohnt sich nur bei sehr schlechter Bildqualität oder Handschrift.

### Warum Basic LLM Chain statt AI Agent?
Für einen einzelnen LLM-Call braucht man keinen Agenten. Basic LLM Chain ist einfacher, schneller, günstiger.

### Warum Move to Processed am Ende?
Verhindert Doppelverarbeitung ohne Tracking-Logik — beim nächsten Lauf listet List Files nur Dateien die noch im Eingangsordner liegen.

---

## 💡 Tipps

**keepSource: both** — Im Extract from File Node unter Options. Ohne das verschwindet das Binary und Analyze Image wirft: `The item has no binary field 'data'`.

**Stop and Error → Continue Regular Output** — In den Node-Settings, nicht in den Parameters. Ohne das bricht der gesamte Workflow bei einer Print-PDF ab statt weiterzumachen.

**Loop-Rückpfeil** — Move to Processed muss mit **Input 0** des Loop verbunden werden. Input 1 ist der interne Weiterloopen-Ausgang, Input 0 ist der Eingang für neue Items von außen.

**`?.` und `??`** — `$json.output?.rechnungsnummer` wirft keinen Fehler wenn `output` nicht existiert. `??` nimmt den rechten Wert wenn links `null` oder `undefined`.

**Analyze Image: Input Type Base64** — Nicht die Default-Einstellung. Muss manuell auf `Base64` umgestellt werden damit das Binary korrekt übergeben wird.

**Execute Step** — Nach Extract from File prüfen ob `$json.text` sichtbar ist UND ob Binary-Tab noch da ist. Nach Edit Fields prüfen ob alle Felder ohne `.output` ankommen.

---

*n8n Training · Files & Dokumentenverarbeitung · Lösung-JSON für Facilitator vorhanden 🚀*

---

# 🧾 Rechnungs-Automatisierung Challenge

**01** Schedule Trigger – alle 5 Minuten
**02** Google Drive List Files
**03** Loop Over Items – Batch Size 1
**04** Google Drive Download
**05** IF – PDF oder Bild? (`fileExtension = pdf`)
**06** Extract from File – `keepSource: both` ⚠️
**07** IF – Digital oder Scan? (`text.length >= 50`)
**08a** Information Extractor + GPT-4o-mini (true)
**08b** Stop and Error (false) → `Continue Regular Output`
**08c** Analyze Image – `inputType: base64` (Bild-Zweig)
**08d** Code – JSON parsen
**09** Edit Fields – `?.` und `??` normalisieren
**10** Google Sheets
**11** Basic LLM Chain + Gemini
**12** Gmail
**13** Google Drive Move

⚠️ **Move → Loop Over Items INPUT 0 verbinden!**

