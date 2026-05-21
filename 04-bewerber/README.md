# Use Case 04 – Bewerbungsscreening: Unterlagen automatisch auswerten

## Szenario

HR lädt Bewerbungsunterlagen (CV, Anschreiben, Zeugnisse) in einen Google Drive Ordner hoch — einzeln oder als Batch. Der Workflow liest die PDFs automatisch aus, vergleicht das Profil mit der aktuellen Stellenbeschreibung von der Website, bewertet Stärken und Schwächen, wertet Arbeitszeugnisse aus, erkennt Auffälligkeiten und schreibt eine strukturierte Empfehlung ins Kandidaten-Tracking-Sheet.

---

## Gesamtprozess (Ziel)

```
Google Drive Trigger (neues PDF im Ordner)
        ↓
PDF(s) herunterladen
        ↓
Text aus PDF extrahieren
        ↓
Stellenbeschreibung von Website scrapen
        ↓
KI-Analyse:
  → Stärken / Schwächen vs. Stelle
  → Arbeitszeugnis dekodieren (Zeugnissprache → Bewertung)
  → Auffälligkeiten erkennen
    (Lücken, kurze Stationen, fehlende Abschlüsse)
        ↓
Gesamt-Score berechnen (0–100%)
        ↓
        ├── Score < 40%
        │       → Absage empfohlen
        │
        └── Score ≥ 40%
                → Weiterleitung an HR empfohlen
        ↓
Eintrag ins Google Sheet Kandidaten-Tracker
```

---

## Google Sheets – Kandidaten-Tracker

| Spalte | Beschreibung |
|---|---|
| Eingang | Datum des Uploads |
| Name | Aus CV extrahiert |
| E-Mail | Aus CV extrahiert |
| Stelle | Aus Dateiname oder Ordner |
| Score | 0–100% |
| Stärken | Passt zur Stelle |
| Schwächen | Fehlt im Profil |
| Zeugnis-Bewertung | Dekodierte Note aus Zeugnissprache (sehr gut / gut / befriedigend / ausreichend) |
| Auffälligkeiten | Lücken, kurze Stationen etc. |
| Empfehlung | Weiterleitung HR / Absage empfohlen |
| Datei-Link | Google Drive Link zur Originaldatei |

---

## Stufe 1 – Text aus einer PDF-Bewerbung lesen 🟢 Pflicht

**Ziel:** Eine Bewerbungs-PDF laden und den Rohtext im n8n Panel sehen.

**Vereinfachung:** Manueller Trigger. Keine KI, keine Analyse. Nur Text extrahieren.

### Testdaten

Es stehen **5 vollständige Bewerbungen** zur Verfügung, je Kandidat 3 Dateien (CV, Anschreiben, Zeugnis):

| Kandidat | CV | Anschreiben | Zeugnis |
|---|---|---|---|
| Braun, Felix | `Braun_Felix_CV.pdf` | `Braun_Felix_Anschreiben.pdf` | `Braun_Felix_Zeugnis.pdf` |
| Hoffmann, Lars | `Hoffmann_Lars_CV.pdf` | `Hoffmann_Lars_Anschreiben.pdf` | `Hoffmann_Lars_Zeugnis.pdf` |
| Koch, Miriam | `Koch_Miriam_CV.pdf` | `Koch_Miriam_Anschreiben.pdf` | `Koch_Miriam_Zeugnis.pdf` |
| Schneider, Anna | `Schneider_Anna_CV.pdf` | `Schneider_Anna_Anschreiben.pdf` | `Schneider_Anna_Zeugnis.pdf` |
| Weber, Jonas | `Weber_Jonas_CV.pdf` | `Weber_Jonas_Anschreiben.pdf` | `Weber_Jonas_Zeugnis.pdf` |

Basis-URL:
```
https://raw.githubusercontent.com/sacpluz/n8n-bit-public/main/bewerber/
```

Vollständiges Beispiel für Weber, Jonas:
```
https://raw.githubusercontent.com/sacpluz/n8n-bit-public/main/bewerber/Weber_Jonas_CV.pdf
https://raw.githubusercontent.com/sacpluz/n8n-bit-public/main/bewerber/Weber_Jonas_Anschreiben.pdf
https://raw.githubusercontent.com/sacpluz/n8n-bit-public/main/bewerber/Weber_Jonas_Zeugnis.pdf
```

Jedes Team wählt einen eigenen Kandidaten.

### Aufgaben

**01 · Manueller Trigger**
Startet den Workflow manuell.

**02 · PDFs laden**

Wählt einen der beiden Wege:

---

**Weg A – HTTP Request** *(empfohlen für den Einstieg)*

Legt für jede der drei Dateien einen eigenen HTTP Request Node an:

- **Method:** `GET`
- **URL:** Raw-URL der jeweiligen Datei (s. Tabelle oben)
- **Response Format:** `File`

Verbindet alle drei Nodes nacheinander oder parallel — der Text aller drei PDFs wird in Stufe 3 zusammengeführt.

> Kein Google Account nötig. Perfekt um den Workflow schnell zum Laufen zu bringen.

---

**Weg B – Google Drive** *(produktionsnah)*

Google Drive Node → Operation: `download`

Wählt eine Beispiel-Bewerbung aus dem Übungsordner in Google Drive.

> Entspricht dem echten Prozess wenn HR Unterlagen in Drive ablegt.

---

**03 · Extract from File Node**
Extrahiert den Text aus jeder PDF-Datei.
Operation: `pdf`

**04 · Set Node**
Speichert den extrahierten Text in saubere Felder:
- `cv_text`
- `anschreiben_text`
- `zeugnis_text`

> Falls ihr mit Weg A arbeitet und drei separate HTTP Request Nodes habt, führt die Texte in einem Merge Node zusammen bevor ihr weiter macht.

### Erwartetes Ergebnis

Der vollständige Text aller drei Bewerbungs-PDFs ist im Output sichtbar — Name, Kontakt, Werdegang, Zeugnis-Formulierungen, alles als Klartext. Das ist die Basis für alle weiteren Schritte.

---

## Stufe 2 – Stellenbeschreibung bereitstellen 🟢 Pflicht

**Ziel:** Die Stellenbeschreibung im Workflow verfügbar machen.

**Hinweis:** Die BridgingIT Karriereseite lädt Stellenanzeigen via JavaScript — ein einfacher HTTP Request liefert nur leeres HTML. Für das Training speichern wir die Stellenbeschreibung daher direkt in Google Sheets. In Stufe 6 (Bonus) kommt der automatische Abruf via Headless Browser.

### Aufgaben

**05 · Google Sheets Node**
Liest die Stellenbeschreibung aus dem Sheet „Stellenbeschreibungen".
Operation: `lookup` nach Stellenbezeichnung `AI Engineer`

> Das Sheet enthält den vollständigen Text der Stelle als einzeiliges Feld `beschreibung`. Einmalig manuell eingepflegt — der Workflow liest sie bei jeder Ausführung frisch.

**06 · Set Node**
Speichert den Text in `stellen_text` damit er in Stufe 3 verfügbar ist.

### Erwartetes Ergebnis

Die Stellenbeschreibung steht als sauberer Text im Workflow und wird mit der Bewerbung zusammen an die KI übergeben.

---

## Stufe 3 – KI-Analyse: Profil vs. Stelle 🟢 Pflicht

**Ziel:** Die KI vergleicht Bewerbungstext und Stellenbeschreibung und liefert eine strukturierte Auswertung.

**Vereinfachung:** Erstmal nur die KI-Ausgabe sehen. Noch kein Score, noch kein Sheet.

### Aufgaben

**07 · Basic LLM Chain Node**
Schickt Bewerbungstext und Stellenbeschreibung an ein LLM.

Prompt-Beispiel:
```
Analysiere die Bewerbung anhand der Stellenbeschreibung und antworte NUR als JSON:

{
  "name": "...",
  "email": "...",
  "staerken": ["...", "..."],
  "schwaechen": ["...", "..."],
  "auffaelligkeiten": ["..."],
  "zeugnis": {
    "aussteller": "...",
    "zeitraum": "...",
    "bewertung": "sehr gut | gut | befriedigend | ausreichend",
    "formel": "Originalformel aus dem Zeugnis",
    "auffaelligkeiten": "..."
  },
  "score": 0-100,
  "begruendung": "..."
}

Hinweis Zeugnissprache: Dekodiere deutsche Arbeitszeugnis-Formulierungen:
- "stets zu vollsten Zufriedenheit" = sehr gut
- "stets zu voller Zufriedenheit" = gut
- "zu voller Zufriedenheit" = gut bis befriedigend
- "zu unserer Zufriedenheit" = befriedigend
- "im Großen und Ganzen" = ausreichend

Stellenbeschreibung:
{{ $json.stellen_text }}

Bewerbung CV:
{{ $json.cv_text }}

Anschreiben:
{{ $json.anschreiben_text }}

Arbeitszeugnis:
{{ $json.zeugnis_text }}
```

**08 · Code Node**
Parst die JSON-Antwort der KI sicher:
```javascript
const raw = $json.text;
const parsed = JSON.parse(raw.replace(/```json|```/g, '').trim());
return [{ json: parsed }];
```

### Erwartetes Ergebnis

Strukturiertes JSON mit Name, E-Mail, Stärken, Schwächen, Zeugnis-Bewertung, Auffälligkeiten und Score. Die KI hat die deutsche Zeugnissprache dekodiert — z.B. „zu unserer Zufriedenheit" → befriedigend.

---

## Stufe 4 – Entscheidung und Google Sheet befüllen 🟢 Pflicht

**Ziel:** Anhand des Scores entscheiden und das Ergebnis in den Kandidaten-Tracker schreiben.

### Aufgaben

**09 · Set Node**
Setzt die Empfehlung anhand des Scores:
```javascript
empfehlung: $json.score >= 40 ? "✅ Weiterleitung HR" : "❌ Absage empfohlen"
```

**10 · Google Sheets Node**
Schreibt alle Felder als neue Zeile in den Kandidaten-Tracker.
Operation: `append`

Mapping:
| Sheet-Spalte | n8n Feld |
|---|---|
| Eingang | `{{ $now.format('DD.MM.YYYY') }}` |
| Name | `{{ $json.name }}` |
| E-Mail | `{{ $json.email }}` |
| Score | `{{ $json.score }}%` |
| Stärken | `{{ $json.staerken.join(', ') }}` |
| Schwächen | `{{ $json.schwaechen.join(', ') }}` |
| Zeugnis-Bewertung | `{{ $json.zeugnis.bewertung }}` |
| Zeugnis-Formel | `{{ $json.zeugnis.formel }}` |
| Auffälligkeiten | `{{ $json.auffaelligkeiten.join(', ') }}` |
| Empfehlung | `{{ $json.empfehlung }}` |

### Erwartetes Ergebnis

Jede gescreente Bewerbung landet als saubere Zeile im Sheet — mit Score, Empfehlung und allen relevanten Infos auf einen Blick.

---

## Stufe 5 – Mehrere Dateien auf einmal verarbeiten 🟡 Bonus

**Ziel:** Alle 5 Kandidaten nacheinander verarbeiten statt einen nach dem anderen manuell zu starten.

### Aufgaben

**11 · Code Node – Dateiliste generieren**
Erstellt die Liste aller 15 Dateien als Array und gibt jeden Kandidaten als Item aus:

```javascript
const kandidaten = ['Braun_Felix', 'Hoffmann_Lars', 'Koch_Miriam', 'Schneider_Anna', 'Weber_Jonas'];
const base = 'https://raw.githubusercontent.com/sacpluz/n8n-bit-public/main/bewerber/';

return kandidaten.map(k => ({ json: {
  kandidat: k,
  cv_url:           `${base}${k}_CV.pdf`,
  anschreiben_url:  `${base}${k}_Anschreiben.pdf`,
  zeugnis_url:      `${base}${k}_Zeugnis.pdf`
}}));
```

**12 · Loop Over Items**
Verarbeitet jeden Kandidaten einzeln durch die Stufen 1–4.

**13 · HTTP Request Nodes** *(3×)*
Laden CV, Anschreiben und Zeugnis anhand der URLs aus dem vorherigen Node:
```
URL: {{ $json.cv_url }}
```

---

## Stufe 6 – Vollautomatisch per Google Drive Trigger 🟡 Bonus

**Ziel:** Der Workflow startet automatisch sobald eine neue PDF im Ordner erscheint.

### Aufgaben

**14 · Google Drive Trigger**
Ersetzt den manuellen Trigger.
Überwacht den Ordner `Bewerbungen/Eingang`.
Auslöser: `Neue Datei`

**15 · Filter Node**
Stellt sicher dass nur PDF-Dateien verarbeitet werden.
```
Bedingung: {{ $json.mimeType }} == "application/pdf"
```

---

## Tipps

**Drei PDFs zusammenführen** — Wenn ihr mit Weg A arbeitet und drei separate HTTP Request Nodes habt, verbindet alle drei mit einem Merge Node bevor ihr den Extract from File Node anschließt. Alternativ: alle drei URLs nacheinander in einer Loop verarbeiten und die Texte per Code Node zusammenführen.

**Zeugnissprache dekodieren** — Deutsche Arbeitszeugnisse verwenden codierte Formulierungen. Gebt dem LLM die Dekodierungstabelle direkt im Prompt mit — wie im Beispiel oben. Ohne diesen Hinweis interpretiert die KI "zu unserer Zufriedenheit" oft fälschlicherweise als positiv.

**Score-Kalibrierung** — Der KI-Score ist ein Richtwert. Testet mit mehreren Kandidaten ob die 40%-Schwelle sinnvoll ist oder ob ihr sie anpassen müsst.

**JSON aus KI absichern** — Wenn die KI kein valides JSON zurückgibt bricht der Workflow ab. Baut einen `try/catch` im Code Node ein oder nutzt den Auto-fixing Output Parser.

**Datenschutz** — Bewerbungsunterlagen sind besonders sensibel. Stellt sicher dass der Google Drive Ordner nur für autorisierte HR-Personen freigegeben ist und keine Daten in Logs gespeichert werden.

---

*n8n Training · Use Case 04 · Bewerbungsscreening · Stufen 1–4 Pflicht · Stufen 5–6 Bonus*
