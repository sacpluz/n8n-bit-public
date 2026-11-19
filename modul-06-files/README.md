# Modul 06 – Files

Dieses Handout ist der Text der Sticky Notes aus dem Workflow, damit ihr ihn im zweiten Browser-Tab lesen und Code kopieren könnt.

**Workflow-Datei:** [`modul-06-files.json`](modul-06-files.json)

**Erklärungen zum Modul** (was der Trainer am Beamer zeigt, zum Nachlesen): [TRAINER.md](TRAINER.md)

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
  → Basic LLM Chain (Azure, gpt-5-mini)
  → Gmail
  → Move to Processed
  → [zurück zu Loop]
```

---

## Aufgaben

### 01 · SCHEDULE TRIGGER — Workflow alle 5 Minuten starten

| Einstellung | Wert |
|---|---|
| Field | `Minutes` |
| Every | `5` |

---

### 02 · GOOGLE DRIVE LIST FILES — Ordnerinhalt abrufen

| Einstellung | Wert |
|---|---|
| Resource | `File/Folder` |
| Operation | `Get Many` |
| Filter | Ordner `Rechnungen n8n` |
| Options → Fields | `*` |

---

### 03 · LOOP OVER ITEMS — Nacheinander verarbeiten

| Einstellung | Wert |
|---|---|
| Batch Size | `1` |

⚠️ Move to Processed am Ende zurück zu **Input 0** des Loop verbinden!

---

### 04 · GOOGLE DRIVE DOWNLOAD — Datei laden

| Einstellung | Wert |
|---|---|
| Operation | `Download` |
| File ID | `={{ $json.id }}` |
| Options → File Name | `={{ $json.name }}` |

---

### 05 · IF: PDF ODER BILD? — Dateityp prüfen

| Einstellung | Wert |
|---|---|
| Bedingung | `{{ $json.fileExtension }}` equals `pdf` |
| Ausgang *true* | Extract from File |
| Ausgang *false* | Analyze Image |

---

### 06 · EXTRACT FROM FILE — Text extrahieren *(PDF-Zweig)*

| Einstellung | Wert |
|---|---|
| Operation | `PDF` |
| Options → Keep Source | **Both** ⚠️ wichtig! |

Warum? Damit das Binary erhalten bleibt falls Analyze Image es später braucht.

---

### 07 · IF: DIGITAL ODER SCAN? — Textinhalt prüfen

| Einstellung | Wert |
|---|---|
| Bedingung | `{{ $json.text.trim().length }}` >= `50` (Typ: Number) |
| Ausgang *true* | Information Extractor |
| Ausgang *false* | Stop and Error |

---

### 08a · INFORMATION EXTRACTOR — Felder aus Text *(digitale PDFs)*

Verbinde **OpenAI Chat Model GPT-4o-mini** als Subnode.

| Einstellung | Wert |
|---|---|
| Text | `{{ $json.text }}` |

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

| Einstellung | Wert |
|---|---|
| Error Message | `Das PDF ist kein digitales, sondern ein Print. Das Bild muss zu einem Bild konvertiert werden.` |
| Settings → On Error | **Continue Regular Output** |

Warum `Continue Regular Output`? Damit der Loop nicht abbricht sondern mit der nächsten Datei weitermacht.

---

### 08c · ANALYZE IMAGE — Bild analysieren *(Bild-Zweig)*

| Einstellung | Wert |
|---|---|
| Model | `GPT-4o-mini` |
| Resource | `Image` |
| Operation | `Analyze` |
| Input Type | `Base64` |

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

| Einstellung | Wert |
|---|---|
| Operation | `Append` |
| Sheet | `Rechnungen` |
| Alle Felder | `{{ $json.rechnungsnummer }}` etc. *(kein `.output` dank Edit Fields)* |
| Dateiname | `={{ $('Google Drive – Download').item.json.name }}` |
| Verarbeitet am | `={{ new Date().toLocaleString('de-DE', {timeZone: 'Europe/Berlin'}) }}` |

---

### 11 · BASIC LLM CHAIN — E-Mail-Text generieren

Verbinde **Azure AI Foundry Chat Model** (`gpt-5-mini`) als Subnode.

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

| Einstellung | Wert |
|---|---|
| Subject | `✅ Rechnung: {{ $('Google Sheets2').item.json.Rechnungsnummer }} - {{ $('Google Sheets2').item.json.Lieferant }}` |
| Message | `{{ $json.text }}` |

---

### 13 · GOOGLE DRIVE MOVE — Datei verschieben

| Einstellung | Wert |
|---|---|
| Operation | `Move` |
| File ID | `={{ $('Google Drive – List Files').item.json.id }}` |
| Folder | `Rechnungen verarbeitet` |

⚠️ Danach zurück zu **Loop Over Items (Input 0)** verbinden!

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
**11** Basic LLM Chain + Azure
**12** Gmail
**13** Google Drive Move

⚠️ **Move → Loop Over Items INPUT 0 verbinden!**
