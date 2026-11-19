# Modul 2 – Data 2: Datenqualität

| ⏱ Dauer | 20 Minuten | 4 Nodes | 👥 Teams à 2 Personen |
|---|---|---|---|

**Workflow-Datei:** [`modul-02-data-2-challenge.json`](modul-02-data-2-challenge.json)

> 💡 Handout im **zweiten Browser-Tab** offen lassen.

---

## 🎯 Szenario

Euer Newsletter-Verteiler ist ein Chaos: doppelte Einträge, inaktive Abonnenten, keine Ordnung. Bereinigt die Liste. Übrig bleiben sollen die **fünf aktivsten** Abonnenten: nur aktive, jede E-Mail-Adresse nur einmal, sortiert nach Öffnungen pro Monat.

**100 rohe Einträge → 4 Nodes → 5 bereinigte Abonnenten**

Ein Eintrag sieht so aus:

```json
{ "name": "Frank Hoffmann", "email": "frank@example.com", "active": true, "signup_date": "2024-06-01", "opens_per_month": 15 }
```

## So arbeitet ihr

1. **Playground** (links): die vier Nodes einzeln mit **Execute step** ausprobieren.
2. **Challenge** (rechts): Nodes sind verbunden, ihr konfiguriert sie.
3. Einmal **Execute workflow**, dann jeden Node mit **Execute step** testen. Achtet auf die **Item-Anzahl** im Output.
4. Tragt in **AUFGABE 0** euren Namen ein. Daraus liest das Scoreboard, wer abgegeben hat.

---

## Aufgaben

### 00 · SET – Namen eintragen

Der erste Node nach dem Start. Hier steht euer Name fürs Scoreboard.

| Name | Typ | Wert |
|---|---|---|
| name | String | euer Vor- und Nachname, z. B. `Anna Müller` |

Kontrolle: Output zeigt `name` mit eurem Namen. Ist das Feld leer, meldet `Ergebnis prüfen` „AUFGABE 0: Euer Name fehlt“ und es landet nichts im Scoreboard.

### 01 · REMOVE DUPLICATES – Duplikate entfernen

| Einstellung | Wert |
|---|---|
| Operation | **Remove Items Repeated Within Current Input** |
| Compare | **Selected Fields** |
| Fields To Compare | `email` |

### 02 · FILTER – Nur aktive Abonnenten

| Einstellung | Wert |
|---|---|
| Value 1 | `{{ $json.active }}` |
| Typ | **Boolean** |
| Bedingung | *is true* |

### 03 · SORT – Die aktivsten zuerst

Zwei Sortierfelder, in dieser Reihenfolge:

| # | Field Name | Order |
|---|---|---|
| 1 | `opens_per_month` | **Descending** |
| 2 | `signup_date` | **Descending** |

Feld 2 entscheidet bei Gleichstand: neuere Anmeldung zuerst.

### 04 · LIMIT – Top 5 ausgeben

| Einstellung | Wert |
|---|---|
| Max Items | `5` |

### Abgabe

**Execute workflow**. `Ergebnis prüfen` muss `korrekt: ✅ Ja` zeigen.

---

## Erwartetes Ergebnis

| # | name | opens_per_month | signup_date |
|---|---|---|---|
| 1 | Frank Hoffmann | 15 | 2024-06-01 |
| 2 | Clara Weber | 12 | 2024-05-22 |
| 3 | Greta Lange | 9 | 2024-04-18 |
| 4 | Anna Müller | 8 | 2024-03-15 |
| 5 | Xenia Herrmann | 8 | 2022-06-01 |

## 🏆 Wertung

Die schnellste **korrekte** Abgabe gewinnt. Details: [Scoreboard](../scoreboard.md).

## 🔧 Wenn es nicht klappt

| Symptom | Ursache |
|---|---|
| Filter lässt nichts durch | Typ steht auf String statt Boolean |
| Anna und Xenia sind vertauscht | zweites Sortierfeld fehlt oder steht auf Ascending |
| Falsche Namen in den Top 5 | Sortierfelder in falscher Reihenfolge, oder Feldname falsch geschrieben (`opens_per_month`, alles klein) |
| Mehr als 5 Einträge | Limit nicht gesetzt |

💡 **Filter ist kein IF:** Filter hat nur einen Ausgang. Was nicht passt, ist weg.

---

## 🚀 Schon fertig?

1. **Reihenfolge tauschen:** Erst Filter, dann Remove Duplicates. Ändert sich das Ergebnis? Wann wäre die Reihenfolge wichtig?
2. **Zweite Liste:** Die fünf Abonnenten, die am längsten dabei sind (ältestes `signup_date`).
3. **Kennzahl:** Hängt einen Aggregate-Node an und zählt, wie viele aktive, eindeutige Abonnenten es insgesamt gibt.
4. **Reaktivierung:** Baut einen zweiten Zweig mit allen **inaktiven** Abonnenten, die sich 2024 angemeldet haben.
