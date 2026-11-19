# Modul 2 – Data 2: Datenqualität – Erklärungen

Was der Trainer am Beamer erklärt. Die Aufgaben stehen in der [README.md](README.md).

---

## Worum es geht

Daten aus echten Systemen sind selten sauber. Die vier Nodes in diesem Modul sind das Standardwerkzeug zum Aufräumen und kommen in fast jedem Workflow vor, der Listen verarbeitet.

```
100 Einträge ─▶ Remove Duplicates ─▶ Filter ─▶ Sort ─▶ Limit ─▶ 5 Einträge
```

Am Beamer nach jedem Node die **Item-Anzahl** zeigen. Daran sieht man sofort, ob ein Schritt wirkt.

## Ablauf am Beamer (8 min) – Playground links im Workflow

Alle Nodes in der gelben Notiz **Playground – Datenqualität**. Jeden Node öffnen, **Execute step**, Item-Anzahl vergleichen.

| Schritt | Node | Was zeigen |
|---|---|---|
| 1 | **Playground – Beispieldaten** | Rohdaten: Duplikate und inaktive Einträge erkennen lassen |
| 2 | **Playground – Filter** | Nur `active = true` bleibt. Kein zweiter Ausgang, der Rest ist weg |
| 3 | **Playground – Remove Duplicates** | Vergleich nur über `email` |
| 4 | **Playground – Sort** | Nach `signup_date` absteigend. Zweites Sortierfeld live hinzufügen |
| 5 | **Playground – Limit** | Höchstens N Items. Live auf 3 stellen |

Frage an die Gruppe: Was passiert, wenn Limit vor Sort steht?

## Node für Node: Bild, Frage, Fallen

Pro Node: kurz erklären, *Execute step*, **Item-Anzahl vorher und nachher** vergleichen, Frage stellen, auflösen. Die Beispieldaten sind absichtlich schmutzig: aktive und inaktive Abonnenten, dazu Duplikate. Am Ende bleibt eine saubere, sortierte Liste.

### Filter – `Playground – Filter`

**Bild:** Ein Türsteher mit Gästeliste. Wer passt, kommt rein. Es gibt keinen Seiteneingang.

**Frage:** Was ist der Unterschied zwischen Filter und IF?
**Antwort:** IF hat zwei Ausgänge, beide lassen sich weiterverarbeiten. Filter hat einen, der Rest ist weg. Braucht ihr den false-Zweig nicht, ist Filter die aufgeräumtere Wahl.

**Fallen:**
- `"true"` (Text) ist nicht `true` (Boolean). Bei strenger Typprüfung fällt dann alles durch.
- Textvergleiche achten standardmäßig auf Groß-/Kleinschreibung: `Berlin` ≠ `berlin`.
- Bleibt nichts übrig, gibt es keinen Fehler. Der Workflow läuft einfach nicht weiter.

### Remove Duplicates – `Playground – Remove Duplicates`

**Bild:** Auf der Einladungsliste steht dieselbe Person dreimal. Remove Duplicates streicht die Wiederholungen.

**Frage:** Warum vergleicht ihr auf `email` und nicht auf `name`?
**Antwort:** Namen sind nicht eindeutig. Zwei verschiedene Frank Hoffmanns sind zwei Menschen. Immer das Feld nehmen, das einen Datensatz eindeutig macht.

**Fallen:**
- Mehrere Vergleichsfelder heißt: Duplikat nur, wenn **alle** übereinstimmen.
- Behalten wird das **erste** Vorkommen. Wer das neueste behalten will, sortiert vorher.

### Sort – `Playground – Sort`

**Bild:** Ein Stapel Bewerbungen, neu geordnet nach Datum oder Score. Der Inhalt bleibt, nur die Reihenfolge ändert sich.

**Frage:** Warum ist die Reihenfolge vor Limit so wichtig?
**Antwort:** Limit nimmt stur die ersten N. Ohne Sortieren sind das zufällige N, mit Sortieren die neuesten oder die besten.

**Fallen:**
- Datum als Text sortiert alphabetisch. `2024-06-01` (ISO) klappt zufällig, `01.06.2024` nicht.
- Beim Sortieren nach Text kommen Großbuchstaben vor Kleinbuchstaben.

### Limit – `Playground – Limit`

**Bild:** Der Drucker hat nur Papier für 20 Zeilen. Der Rest bleibt im System, kommt aber nicht mit.

**Frage:** Was passiert, wenn Limit vor Sort steht? Und was, wenn Limit vor Filter steht?
**Antwort:** Vor Sort: N beliebige Einträge statt der Top N. Vor Filter: Erst auf 20 kürzen, dann fallen noch welche raus, am Ende sind es weniger als 20. Faustregel: **filtern, sortieren, dann begrenzen.**

**Falle:** Limit ist kein Blättern. Die nächsten 20 bekommt man nicht mit einem zweiten Limit, dafür braucht es Pagination (API-Parameter oder Code).

### Übergang zur Challenge

„Ihr habt gerade aus einer schmutzigen Liste eine saubere, sortierte, eindeutige gemacht, ganz ohne Code. In der Challenge baut ihr so eine Pipeline selbst. Ich starte die Zeit, wenn alle bereit sind.“

## Die Nodes

| Node | Was er macht | Worauf achten |
|---|---|---|
| **Remove Duplicates** | Entfernt Items, die in den gewählten Feldern gleich sind | Nur `email` vergleichen. Bei *All Fields* bleiben Duplikate mit unterschiedlichem Namen oder Datum drin |
| **Filter** | Lässt nur Items durch, die die Bedingung erfüllen. Anders als IF gibt es keinen zweiten Ausgang, der Rest fällt weg | Typ **Boolean**, Bedingung *is true* |
| **Sort** | Sortiert nach einem oder mehreren Feldern | Reihenfolge der Sortierfelder zählt: das zweite Feld entscheidet nur bei Gleichstand im ersten |
| **Limit** | Gibt nur die ersten N Items weiter | Wirkt nur sinnvoll **nach** dem Sortieren |

## Warum die Reihenfolge der Nodes zählt

- Erst **Sort**, dann **Limit**: Andersherum bekommt man fünf zufällige Abonnenten statt der fünf aktivsten.
- **Duplikate und Filter:** Im Workflow kommt Remove Duplicates zuerst. Robuster wäre erst filtern, dann Duplikate entfernen: Sonst kann ein inaktiver Doppel-Eintrag den aktiven verdrängen, und die Person fällt ganz raus. Gute Diskussionsfrage.

Gute Frage an die Gruppe: Was passiert, wenn man Limit vor Sort schaltet? Kurz ausprobieren lassen.

## Typische Fallen

- **Feldnamen klein:** `opens_per_month`, `signup_date`, `active`.
- **Filter auf Text statt Boolean:** `"true"` ist nicht `true`. Mit Typ String fallen alle Einträge durch.
- **Sortierrichtung:** Descending = größter Wert zuerst. Ascending liefert die *inaktivsten* fünf.

## Auflösung

Lösungs-Workflow am Beamer zeigen, dann Scoreboard. Erwartete Reihenfolge der Top 5 steht in der README unter „Erwartetes Ergebnis“.
