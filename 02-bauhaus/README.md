# Use Case 02 – Bauhaus: App-Bewertungen automatisch auswerten

## Szenario

Das Produktteam möchte wissen was Kunden über die Bauhaus-App denken — ohne täglich manuell App Store und Play Store zu checken. Kritische Bewertungen sollen sofort auffallen, positive Bewertungen sollen beantwortet werden, und wiederkehrende Feature-Wünsche sollen direkt als Jira-Ticket landen.

---

## Gesamtprozess (Ziel)

```
Schedule Trigger (täglich)
        ↓
App Store + Play Store Reviews abrufen
        ↓
Duplikate filtern
        ↓
Pro Review: KI-Analyse
  → Sentiment Score + Kategorie
    (Bug / Feature Request / Lob / Beschwerde)
        ↓
        ├── Negativ / ≤ 2 Sterne
        │       → Alert-Mail sofort
        │       → Antwort-Draft (kritisch)
        │
        └── Positiv
                → Antwort-Draft (freundlich)
        ↓
Feature Request erkannt?
        → Jira Ticket erstellen (optional)
        ↓
Alle Reviews ins Google Sheet
        ↓
Freitags: Weekly Digest Mail an Produktteam
```

---

## Google Sheets – Tabellenstruktur

| Spalte | Beschreibung |
|---|---|
| Review ID | Eindeutige ID aus App Store / Play Store |
| Datum | Datum der Bewertung |
| Plattform | App Store / Play Store |
| Sterne | 1–5 |
| Review Text | Originaltext |
| Sentiment | positiv / negativ / neutral |
| Sentiment Score | 0.0–1.0 |
| Kategorie | Bug / Feature Request / Lob / Beschwerde |
| Antwort-Draft | KI-generierter Antwortvorschlag |
| Alert gesendet | ja / nein |
| Jira Ticket | Ticket-URL (wenn erstellt) |

---

## Stufe 1 – Erste Reviews sehen 🟢 Pflicht

**Ziel:** Reviews abrufen und im n8n Panel sichtbar machen.

**Vereinfachung:** Manueller Trigger. Keine KI, kein Sheet.

### Aufgaben

**01 · Manueller Trigger**
Startet den Workflow manuell über „Execute Workflow".

**02 · Reviews laden**

Wählt einen der beiden Wege:

---

**Weg A – App Store RSS Feed** *(empfohlen)*

HTTP Request Node, keine Authentifizierung nötig:

```
URL: https://itunes.apple.com/de/rss/customerreviews/id=1615566534/sortby=mostrecent/json
Methode: GET
```

Danach: **Split Out Node** → Feld: `feed.entry`

> Liefert die aktuellsten echten Reviews direkt aus dem App Store.

---

**Weg B – Bereitgestellte CSV-Datei** *(Fallback)*

Falls der App Store Feed nicht funktioniert, steht eine fertige Review-Sammlung als CSV zur Verfügung:

```
URL: https://raw.githubusercontent.com/sacpluz/n8n-bit-public/main/bauhaus/empfehlungen.csv
Methode: GET
Response Format: Text
```

Danach: **Extract from File Node** (oder **Code Node**) um die CSV-Zeilen in einzelne Items umzuwandeln.

> Die CSV enthält dieselben Felder wie der App Store Feed — der Rest des Workflows bleibt identisch.

---

**03 · Set Node**
Extrahiert die relevanten Felder in saubere Feldnamen — egal welchen Weg ihr gewählt habt.

> → `review_id`, `datum`, `sterne`, `text`, `autor`

### Erwartetes Ergebnis

Jedes Item im Output enthält einen einzelnen Review mit sauberen Feldern. Ihr seht die Kundenstimmen direkt im n8n Panel.

---

## Stufe 2 – Sentiment analysieren und ins Sheet schreiben 🟢 Pflicht

**Ziel:** Jeden Review durch eine KI schicken, Sentiment und Kategorie bestimmen, Ergebnis ins Google Sheet schreiben.

**Vereinfachung:** Noch kein Alert, noch kein Antwort-Draft. Nur analysieren und speichern.

### Aufgaben

**04 · Basic LLM Chain Node**
Schickt den Review-Text an ein LLM. Prompt-Beispiel:

```
Analysiere folgende App-Bewertung und antworte NUR als JSON:
{
  "sentiment": "positiv" | "negativ" | "neutral",
  "score": 0.0-1.0,
  "kategorie": "Bug" | "Feature Request" | "Lob" | "Beschwerde" | "Sonstiges"
}

Bewertung: {{ $json.text }}
```

**05 · Set Node**
Kombiniert Original-Review-Daten mit den KI-Ergebnissen in einem Item.

**06 · Google Sheets Node**
Schreibt jedes Item als neue Zeile ins Sheet „App Reviews".
Operation: `append`

### Erwartetes Ergebnis

Das Google Sheet füllt sich mit Reviews inkl. Sentiment und Kategorie. Ihr seht auf einen Blick wie die Stimmungslage ist.

---

## Stufe 3 – Alert bei negativen Reviews 🟢 Pflicht

**Ziel:** Wenn ein Review negativ ist oder ≤ 2 Sterne hat, geht sofort eine Alert-Mail raus.

### Aufgaben

**07 · IF Node**
Prüft ob ein Review kritisch ist.

```
Bedingung A: {{ $json.sterne }} <= 2
ODER
Bedingung B: {{ $json.sentiment }} == "negativ"
```

**08 · Gmail / Send Email Node** *(true-Zweig)*
Sendet eine Alert-Mail ans Produktteam.

Inhalt der Mail:
```
⚠️ Neue kritische App-Bewertung

Plattform: {{ $json.plattform }}
Sterne: {{ $json.sterne }}
Kategorie: {{ $json.kategorie }}

"{{ $json.text }}"
```

**09 · Testen**
Führt den Workflow aus. Prüft ob kritische Reviews eine Mail auslösen und neutrale/positive nicht.

### Erwartetes Ergebnis

Ihr erhaltet eine E-Mail sobald ein Kunde weniger als 3 Sterne gibt oder das Sentiment negativ ist — egal ob ihr gerade am Rechner seid.

---

## Stufe 4 – Antwort-Draft generieren 🟡 Bonus

**Ziel:** Für jeden Review — egal ob positiv oder negativ — einen passenden Antwortvorschlag generieren und ins Sheet schreiben.

### Aufgaben

**10 · Basic LLM Chain Node**
Generiert einen Antwort-Draft je nach Sentiment.

Prompt-Beispiel:
```
Du bist Kundenbetreuer bei Bauhaus. Schreibe eine kurze, freundliche 
App-Store-Antwort auf folgende Bewertung. 
Ton: {{ $json.sentiment == "positiv" ? "herzlich und dankbar" : "verständnisvoll und lösungsorientiert" }}
Sprache: Deutsch. Max. 3 Sätze.

Bewertung: {{ $json.text }}
```

**11 · Google Sheets Node**
Aktualisiert die bestehende Zeile im Sheet mit dem Antwort-Draft.
Operation: `update` (anhand Review ID)

---

## Stufe 5 – Feature Requests → Jira Ticket 🟡 Bonus

**Ziel:** Wenn ein Review als „Feature Request" kategorisiert wurde, automatisch ein Jira-Ticket anlegen.

### Aufgaben

**12 · IF Node**
Filtert Reviews mit `kategorie == "Feature Request"`.

**13 · HTTP Request Node (Jira API)**
Erstellt ein Ticket in eurem Jira-Projekt.

```
URL: https://EUER-DOMAIN.atlassian.net/rest/api/3/issue
Methode: POST
Auth: Basic Auth (E-Mail + API Token)

Body:
{
  "fields": {
    "project": { "key": "PROJ" },
    "summary": "App Review Feature Request: {{ $json.text | truncate(80) }}",
    "description": "{{ $json.text }}\n\nQuelle: {{ $json.plattform }} | {{ $json.datum }}",
    "issuetype": { "name": "Story" }
  }
}
```

**14 · Google Sheets Node**
Schreibt die Jira-Ticket-URL in die entsprechende Spalte des Sheets.

---

## Stufe 6 – Play Store + Schedule + Weekly Digest 🟡 Bonus

**Ziel:** Vollautomatisch. Täglich laufen, beide Plattformen, freitags Weekly Digest.

### Aufgaben

**15 · Schedule Trigger**
Ersetzt den manuellen Trigger. Täglich um 08:00 Uhr.

**16 · Play Store via Apify**
Apify hat einen fertigen Play Store Scraper mit Free Tier.
HTTP Request Node → Apify API → Reviews als JSON.

**17 · Duplikat-Check**
Google Sheets Lookup: Existiert die Review ID bereits?
Nur neue Reviews werden verarbeitet.

**18 · Weekly Digest (freitags)**
Zweiter Schedule Trigger (freitags 17:00) → Sheet auslesen →
KI generiert Zusammenfassung → Mail an Produktteam.

Digest enthält:
- Durchschnittliche Sternebewertung der Woche
- Top 3 Beschwerden
- Top 3 Feature Requests
- Stimmungstrend vs. Vorwoche

---

## Tipps

**App Store RSS Limit** — der Feed liefert max. 50 Reviews. Für ältere Reviews braucht ihr die offizielle Apple Search API (kostenpflichtig). Für das Training reicht der RSS Feed.

**CSV Fallback** — die bereitgestellte CSV hat dieselbe Struktur wie der App Store Feed. Wenn ihr auf Weg B umsteigt, müsst ihr nur den ersten Node tauschen — der Rest des Workflows bleibt identisch.

**JSON Parsing** — die KI-Antwort kommt als Text. Nutzt `JSON.parse()` im Code Node um sie sicher in ein Objekt umzuwandeln bevor ihr auf die Felder zugreift.

**Duplikate vermeiden** — Review IDs aus dem App Store sind stabil. Speichert sie im Sheet und prüft vor dem Verarbeiten ob die ID bereits existiert.

**Jira API Token** — erstellt euch unter `id.atlassian.net/manage-profile/security/api-tokens` einen API Token. Niemals direkt im Workflow hardcoden — immer als n8n Credential speichern.

**Sentiment Schwellenwert** — testet mit echten Reviews ob 0.4 als Score-Grenze für "negativ" sinnvoll ist oder ob ihr ihn anpassen müsst.

---

*n8n Training · Use Case 02 · Bauhaus App Reviews · Stufen 1–3 Pflicht · Stufen 4–6 Bonus*
