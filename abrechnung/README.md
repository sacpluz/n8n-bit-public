# Use Case 03 – Stundenabrechnung: Automatische Rechnungserstellung

## Szenario

Berater:innen erfassen ihre Stunden monatlich in Google Sheets. Am Monatsende prüft der Workflow automatisch ob die Einträge plausibel sind, vergleicht Ist- mit Forecast-Stunden und entscheidet: Abweichung zu groß oder Auffälligkeit gefunden → Mail an Berater:in. Alles in Ordnung → Rechnung als PDF generieren und an den Kundenansprechpartner schicken.

---

## Gesamtprozess (Ziel)

```
Schedule Trigger (1. des Monats)
        ↓
Stunden aus Google Sheet lesen
        ↓
KI: Plausibilitätsprüfung der Einträge
  → Vage Beschreibungen? Zu viele Stunden pro Tag?
    Wochenendeinträge? Passt zur Projekttätigkeit?
        ↓
Ist-Stunden vs. Forecast berechnen
        ↓
        ├── Auffälligkeit ODER Abweichung > 10%
        │       → Mail an Berater:in zur Klärung
        │
        └── Alles plausibel, Abweichung ≤ 10%
                → PDF-Rechnung generieren
                → Mail mit Rechnung an Kundenansprechpartner
```

---

## Google Sheets – Struktur

### Sheet 1: Stundenerfassung *(von Berater:innen befüllt)*

| Spalte | Beschreibung | Beispiel |
|---|---|---|
| Datum | Tag der Leistung | 03.01.2026 |
| Berater | Name | Müller, Thomas |
| Projekt | Projektname | Bauhaus Digital |
| Tätigkeit | Beschreibung der Arbeit | API-Integration Backend |
| Stunden | Geleistete Stunden | 7,5 |

### Sheet 2: Forecast *(Projektplanung)*

| Spalte | Beschreibung | Beispiel |
|---|---|---|
| Berater | Name | Müller, Thomas |
| Projekt | Projektname | Bauhaus Digital |
| Monat | Abrechnungsmonat | Januar 2026 |
| Forecast Stunden | Geplante Stunden | 160 |

### Sheet 3: Kundenstamm *(Referenzdaten)*

| Spalte | Beschreibung | Beispiel |
|---|---|---|
| Projekt | Projektname | Bauhaus Digital |
| Kundenname | Vollständiger Name | Bauhaus AG |
| Ansprechpartner | Name | Max Mustermann |
| E-Mail | Rechnungsempfänger | m.mustermann@bauhaus.de |
| Stundensatz | EUR netto | 150,00 |

---

## KI Plausibilitätsprüfung – was wird geprüft?

| Kriterium | Beispiel für Auffälligkeit |
|---|---|
| Vage Beschreibung | „Meeting", „Sonstiges", „Diverses" für viele Stunden |
| Stunden pro Tag | Mehr als 10 Stunden an einem Tag |
| Wochenendeinträge | Samstag / Sonntag ohne Erklärung |
| Gesamtplausibilität | 22 Arbeitstage × 8h = max. 176h — mehr als das ist verdächtig |
| Projektbezug | Tätigkeitsbeschreibung passt nicht zum Projekt |

---

## Stufe 1 – Stunden aus dem Sheet lesen 🟢 Pflicht

**Ziel:** Monatliche Stundeneinträge eines Beraters aus Google Sheets lesen und im n8n Panel sehen.

**Vereinfachung:** Manueller Trigger. Nur ein Berater, ein Projekt. Keine KI, keine Mail.

### Aufgaben

**01 · Manueller Trigger**
Startet den Workflow manuell.

**02 · Google Sheets Node**
Liest alle Zeilen aus dem Sheet „Stundenerfassung".
Operation: `getAll`

Filtert auf den aktuellen Monat und einen festen Berater zum Testen.

**03 · Aggregate Node**
Fasst alle Einträge des Beraters zusammen.
Berechnet die Gesamtstunden: Summe aller Stunden-Felder.

**04 · Set Node**
Bereitet die Kerndaten vor:
> → `berater`, `projekt`, `monat`, `ist_stunden`, `anzahl_eintraege`

### Erwartetes Ergebnis

Ein einzelnes Item mit den aggregierten Stunden des Beraters für den Monat. Ihr seht auf einen Blick: wer hat wie viele Stunden auf welchem Projekt gebucht.

---

## Stufe 2 – KI Plausibilitätsprüfung 🟢 Pflicht

**Ziel:** Die einzelnen Stundeneinträge durch eine KI schicken und Auffälligkeiten erkennen lassen.

**Vereinfachung:** Nur die KI-Ausgabe im Panel sehen — noch keine Entscheidungslogik.

### Aufgaben

**05 · Basic LLM Chain Node**
Schickt alle Einträge des Beraters an ein LLM. Prompt-Beispiel:

```
Du bist Rechnungsprüfer in einem IT-Beratungsunternehmen.
Prüfe folgende Stundeneinträge auf Plausibilität.
Antworte NUR als JSON:
{
  "plausibel": true | false,
  "auffaelligkeiten": ["..."],
  "empfehlung": "freigeben" | "klaeren"
}

Einträge:
{{ $json.eintraege }}
```

**06 · Set Node**
Kombiniert KI-Ergebnis mit den aggregierten Stunden aus Stufe 1.

### Erwartetes Ergebnis

Das Item enthält jetzt zusätzlich: `plausibel`, `auffaelligkeiten`, `empfehlung`. Ihr seht was die KI als verdächtig markiert hat.

---

## Stufe 3 – Abweichung berechnen und entscheiden 🟢 Pflicht

**Ziel:** Ist-Stunden mit Forecast vergleichen. Entscheiden ob die Abrechnung freigegeben wird oder zur Klärung geht.

### Aufgaben

**07 · Google Sheets Node**
Liest den Forecast für diesen Berater und dieses Projekt aus Sheet 2.
Operation: `lookup`

**08 · Code Node**
Berechnet die Abweichung:

```javascript
const ist = $json.ist_stunden;
const forecast = $json.forecast_stunden;
const abweichung = Math.abs((ist - forecast) / forecast * 100);

return [{
  json: {
    ...$json,
    forecast_stunden: forecast,
    abweichung_prozent: Math.round(abweichung * 10) / 10,
    freigabe: abweichung <= 10 && $json.empfehlung === "freigeben"
  }
}];
```

**09 · IF Node**
Entscheidet anhand des `freigabe`-Feldes:
- `true` → Rechnung erstellen
- `false` → Klärungsmail an Berater:in

### Erwartetes Ergebnis

Der Workflow trifft automatisch die richtige Entscheidung. Testet mit einem Eintrag der > 10% Abweichung hat und einem der freigegeben wird.

---

## Stufe 4 – Klärungsmail an Berater:in 🟢 Pflicht

**Ziel:** Bei Auffälligkeit oder zu großer Abweichung eine strukturierte Mail an die Berater:in schicken.

### Aufgaben

**10 · Gmail / Send Email Node** *(false-Zweig)*

Betreff: `Stundenabrechnung {{ $json.monat }} – Bitte prüfen`

Inhalt:
```
Hallo {{ $json.berater }},

bei deiner Stundenabrechnung für {{ $json.projekt }} im {{ $json.monat }}
gibt es folgende Auffälligkeiten:

{{ $json.auffaelligkeiten }}

Ist-Stunden: {{ $json.ist_stunden }}h
Forecast: {{ $json.forecast_stunden }}h
Abweichung: {{ $json.abweichung_prozent }}%

Bitte melde dich kurz damit wir die Abrechnung freigeben können.
```

---

## Stufe 5 – Rechnung generieren und versenden 🟡 Bonus

**Ziel:** Bei freigegebener Abrechnung eine einfache PDF-Rechnung erstellen und an den Kunden schicken.

### Aufgaben

**11 · Google Sheets Node**
Liest Kundendaten (Ansprechpartner, E-Mail, Stundensatz) aus Sheet 3.
Operation: `lookup` nach Projektname.

**12 · Code Node**
Berechnet den Rechnungsbetrag und generiert ein HTML-Template:

```javascript
const betrag_netto = $json.ist_stunden * $json.stundensatz;
const mwst = betrag_netto * 0.19;
const betrag_brutto = betrag_netto + mwst;

const html = `
<h2>Rechnung – {{ $json.monat }}</h2>
<p>Projekt: {{ $json.projekt }}</p>
<p>Berater: {{ $json.berater }}</p>
<table>
  <tr><td>Stunden</td><td>${$json.ist_stunden} h</td></tr>
  <tr><td>Stundensatz</td><td>${$json.stundensatz} EUR</td></tr>
  <tr><td>Netto</td><td>${betrag_netto.toFixed(2)} EUR</td></tr>
  <tr><td>MwSt. 19%</td><td>${mwst.toFixed(2)} EUR</td></tr>
  <tr><td><b>Brutto</b></td><td><b>${betrag_brutto.toFixed(2)} EUR</b></td></tr>
</table>`;

return [{ json: { ...$json, html, betrag_netto, betrag_brutto } }];
```

**13 · Convert to File Node**
Wandelt das HTML in eine PDF-Datei um.
> Operation: `toText` → Dateiname: `Rechnung_{{ $json.projekt }}_{{ $json.monat }}.pdf`

**14 · Gmail Node**
Sendet die Rechnung als Anhang an den Kundenansprechpartner.

---

## Stufe 6 – Alle Berater automatisch abrechnen 🟡 Bonus

**Ziel:** Der Workflow läuft vollautomatisch für alle Berater und alle Projekte am Monatsende.

### Aufgaben

**15 · Schedule Trigger**
Ersetzt den manuellen Trigger. Läuft am 1. jeden Monats um 06:00 Uhr.

**16 · Split Out Node**
Iteriert über alle Berater-Projekt-Kombinationen aus dem Forecast-Sheet.

**17 · Loop**
Führt die gesamte Prüf- und Abrechnungslogik für jeden Berater einzeln aus.

---

## Tipps

**Monat filtern** — Nutzt einen Code Node um den aktuellen Monat als String zu erzeugen: `new Date().toLocaleDateString('de-DE', { month: 'long', year: 'numeric' })`. Damit filtert ihr die richtigen Zeilen im Sheet.

**JSON aus KI parsen** — Die KI-Antwort kommt als Text. Immer `JSON.parse()` verwenden und in einem Try/Catch absichern damit ein schlechtes JSON-Response den Workflow nicht abbricht.

**Stundensatz nie hardcoden** — Immer aus dem Kundenstamm-Sheet lesen. Dann müsst ihr bei Preisänderungen nur eine Stelle anpassen.

**HTML zu PDF** — Das `Convert to File` Node in n8n erzeugt eine Textdatei. Für ein echtes PDF nutzt einen HTTP Request an einen kostenlosen Dienst wie `html2pdf.app` — sendet das HTML als Body, erhaltet eine PDF-Datei zurück.

---

*n8n Training · Use Case 03 · Stundenabrechnung T&M · Stufen 1–4 Pflicht · Stufen 5–6 Bonus*
