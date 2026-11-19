# Modul 1 – Data 1: Lead Qualification Pipeline

| ⏱ Dauer | 45 Minuten (10 min Playground, 30 min Challenge, 5 min Auflösung) | 6 Nodes | 👥 Teams à 2 Personen |
|---|---|---|---|

**Workflow-Datei:** [`modul-01-data-1-challenge.json`](modul-01-data-1-challenge.json)

> 💡 Handout im **zweiten Browser-Tab** offen lassen. Voraussetzung: [Modul 0](../modul-00-grundlagen/README.md).

---

## 🎯 Szenario

Ihr habt eine Liste von Leads mit einem **Score**. Baut einen Workflow, der sie automatisch einstuft: Leads mit Score ≥ 60 werden als `Qualifiziert` / `Hoch` markiert, der Rest als `Abgelehnt` / `Niedrig`. Am Ende landet alles wieder in einem einzigen Array.

**Input (steckt schon im Node `Leads generieren`):**

```json
{
  "leads": [
    { "name": "Sarah Johnson", "score": 75, "region": "EMEA" },
    { "name": "David Smith",   "score": 45, "region": "AMER" },
    { "name": "Emily Chen",    "score": 88, "region": "APAC" },
    { "name": "Carlos Mendez", "score": 32, "region": "EMEA" },
    { "name": "Anika Patel",   "score": 91, "region": "APAC" }
  ]
}
```

## So arbeitet ihr

1. **Playground** (links im Workflow): Nodes einzeln mit **Execute step** ausführen und Input mit Output vergleichen.
2. **Challenge** (rechts): Die Nodes sind schon verbunden. Ihr konfiguriert sie von links nach rechts.
3. Einmal **Execute workflow** klicken, dann jeden Aufgaben-Node mit **Execute step** testen, bevor ihr weitergeht.
4. Der Node **Ergebnis prüfen** zeigt im Feld `hinweise`, was noch nicht stimmt.
5. Euer Name fürs Scoreboard kommt automatisch aus dem Workflow-Namen. **Nichts eintragen.**

---

## Aufgaben

### 01 · SPLIT OUT – Array in einzelne Items aufteilen

`Leads generieren` liefert **ein** Item mit einem Array im Feld `leads`. Macht daraus fünf Items.

> → Fields To Split Out: `leads`

Kontrolle: Output zeigt **5 items**.

### 02 · IF – Score-Schwelle setzen

Leads mit Score ≥ 60 gehen in den *true*-Zweig, alle anderen in den *false*-Zweig.

> → `{{ $json.score }}` · **Number** · *is greater than or equal to* · `60`

Kontrolle: true = 3 Items, false = 2 Items.

### 03a · SET – Qualifizierte Leads markieren (true-Zweig)

Zwei neue Felder hinzufügen. Die bestehenden Felder müssen erhalten bleiben.

> → `Type` = `Qualifiziert` · `Priority` = `Hoch`
> → **Include Other Input Fields** einschalten

### 03b · SET – Abgelehnte Leads markieren (false-Zweig)

> → `Type` = `Abgelehnt` · `Priority` = `Niedrig`
> → **Include Other Input Fields** einschalten

⚠️ Feldnamen und Werte **exakt** so schreiben, mit großem Anfangsbuchstaben. `type` ist nicht `Type`.

### 04 · MERGE – Beide Zweige zusammenführen

> → Mode: **Append**

Kontrolle: Output zeigt wieder 5 Items.

### 05 · AGGREGATE – Alle Items in ein Array packen

> → Aggregate: **All Item Data (Into a Single List)**
> → Put Output in Field: `processed_leads`

Kontrolle: Output zeigt **1 item** mit dem Array `processed_leads`.

### Abgabe

**Execute workflow** klicken. `Ergebnis prüfen` muss `korrekt: ✅ Ja` zeigen, dann trägt der letzte Node euer Ergebnis ins Scoreboard ein.

---

## Erwartetes Ergebnis

| name | score | region | Type | Priority |
|---|---|---|---|---|
| Sarah Johnson | 75 | EMEA | Qualifiziert | Hoch |
| Emily Chen | 88 | APAC | Qualifiziert | Hoch |
| Anika Patel | 91 | APAC | Qualifiziert | Hoch |
| David Smith | 45 | AMER | Abgelehnt | Niedrig |
| Carlos Mendez | 32 | EMEA | Abgelehnt | Niedrig |

## 🏆 Wertung

Die schnellste **korrekte** Abgabe gewinnt. Korrektheit geht vor Tempo: Falsche Abgaben zählen nicht. Details: [Scoreboard](../scoreboard.md).

## 🔧 Wenn es nicht klappt

| Symptom | Ursache |
|---|---|
| IF lässt nichts durch | Typ steht auf String statt Number, oder das Feld heißt nicht `score` |
| Nach dem Set fehlen `name`, `score`, `region` | *Include Other Input Fields* ist aus |
| `Ergebnis prüfen` meldet „`processed_leads` fehlt" | Feldname in Aufgabe 5 falsch geschrieben oder leer |
| Workflow ist grün, aber `korrekt: ❌ Nein` | `hinweise` lesen. Meist Groß-/Kleinschreibung |
| Merge liefert nur 3 oder 2 Items | Mode steht nicht auf Append |

---

## 🚀 Schon fertig?

1. **Dritte Stufe:** Leads mit Score ≥ 85 bekommen `Priority` = `Sehr hoch`. Tipp: Switch statt IF, oder ein zweites IF. (Für die Wertung zählt die Abgabe davor.)
2. **Region:** Zusätzliches Feld `Team` setzen: `EMEA` → `Team Europa`, sonst `Team International`. Geht mit einer Expression: `{{ $json.region === 'EMEA' ? 'Team Europa' : 'Team International' }}`
3. **Ohne Verzweigung:** Löst die ganze Aufgabe mit einem einzigen Set-Node und zwei Expressions. Was ist lesbarer?
4. Helft dem Nachbarteam beim Debuggen.
