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
