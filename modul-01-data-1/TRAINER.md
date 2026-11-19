# Modul 1 – Data 1: Lead Qualification Pipeline – Erklärungen

Was der Trainer am Beamer erklärt. Die Aufgaben stehen in der [README.md](README.md).

---

## Worum es geht

Ein typisches Muster in n8n: **auffächern → verzweigen → anreichern → zusammenführen → einsammeln**. Wer das einmal gebaut hat, erkennt es in fast jedem Daten-Workflow wieder.

```
1 Item mit Array ─▶ Split Out ─▶ 5 Items ─▶ IF ─┬─ true ─▶ Set ─┐
                                               └─ false ─▶ Set ─┴─▶ Merge ─▶ 5 Items ─▶ Aggregate ─▶ 1 Item
```

Am Beamer nach jedem Node auf die **Item-Anzahl** im Output zeigen: 1 → 5 → 3 + 2 → 5 → 1.

## Items

n8n arbeitet mit **Items**. Jeder Node bekommt eine Liste von Items und läuft für jedes Item einmal. Ein Array *innerhalb* eines Items ist dagegen nur ein Feld. Deshalb muss es erst aufgefächert werden, bevor IF oder Set pro Lead arbeiten können.

## Die Nodes

| Node | Was er macht | Worauf achten |
|---|---|---|
| **Split Out** | Macht aus einem Array-Feld einzelne Items | Feldname exakt (`leads`) |
| **IF** | Teilt Items in zwei Zweige: Bedingung erfüllt (true) oder nicht (false) | Typ **Number**, sonst vergleicht n8n Text (`"75" >= "60"`) |
| **Edit Fields (Set)** | Legt Felder an oder ändert sie | **Include Other Input Fields** an, sonst gehen `name`, `score`, `region` verloren |
| **Merge** | Führt zwei Eingänge zusammen | Modus **Append** hängt einfach hintereinander. Andere Modi verknüpfen Items paarweise und liefern hier falsche Ergebnisse |
| **Aggregate** | Sammelt alle Items wieder in **ein** Item mit einem Array | Zielfeld `processed_leads`, sonst findet `Ergebnis prüfen` nichts |

## Typische Fallen

- **Groß-/Kleinschreibung:** `Type` ist nicht `type`, `Qualifiziert` ist nicht `qualifiziert`. `Ergebnis prüfen` meldet ähnlich geschriebene Felder ausdrücklich.
- **Merge wartet auf beide Eingänge.** Läuft nur ein Zweig, kommt am Merge nichts an. Einmal *Execute workflow* statt einzelner Steps.
- **Expression statt Fixed:** Im IF muss Value 1 auf *Expression* stehen (`{{ $json.score }}`), sonst wird der Text `{{ $json.score }}` verglichen.

## Auflösung

Lösungs-Workflow am Beamer zeigen, dann Scoreboard. Gute Frage an die Gruppe: Was passiert bei Score genau 60? (Richtig: qualifiziert, weil *greater than or equal*.)
