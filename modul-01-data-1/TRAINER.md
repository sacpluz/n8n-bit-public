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

## Ablauf am Beamer (10 min) – Playground links im Workflow

Alle Nodes in der gelben Notiz **Playground**. Jeden Node öffnen, **Execute step**, auf die **Item-Anzahl** im Output zeigen.

| Schritt | Node | Was zeigen |
|---|---|---|
| 1 | **Playground – Leads** | 1 Item, darin ein Array `leads` |
| 2 | **Playground – Split Out** | 5 Items: aus dem Array werden einzelne Items |
| 3 | **Playground – Code statt Split Out** | Dasselbe per Code. Nur zeigen, nicht vertiefen |
| 4 | **Playground – IF** | Bedingung `name` enthält „Sarah“: 1 Item true, 4 false |
| 5 | **Playground – Switch** | Wie IF, aber mit mehreren benannten Ausgängen |
| 6 | **Playground – Aggregate** | Wieder 1 Item, die Leads stecken im Feld `data` |
| 7 | **Playground – IF nach Aggregate** | Nach dem Einsammeln: Zugriff mit `$json.data[0].name` |
| 8 | **Playground – Merge** | *Execute step* auf Merge: **Zweig A** läuft über **Playground – Wait**, **Zweig B** direkt. Merge wartet auf beide und hängt sie zusammen |

Danach die Skizze unten zeigen: genau dieses Muster baut ihr in den Aufgaben.

## Node für Node: Bild, Frage, Fallen

Pro Node: ein, zwei Sätze erklären, alle führen *Execute step* aus und schauen auf den Output, dann die Frage stellen, kurz warten, auflösen. Kein Zeitdruck, die Challenge kommt danach.

### Split Out – `Playground – Split Out`

**Bild:** Ein Umschlag mit fünf Briefen. Split Out öffnet ihn und legt jeden Brief einzeln auf den Tisch. Ab jetzt bearbeitet n8n jeden Brief für sich.

**Frage:** Wann kommen bei euch Daten als Liste an, die ihr einzeln bearbeiten müsst?
**Antwort:** Fast immer, wenn eine API eine Liste liefert: offene Tickets, Produkte, Bestellungen. Viele API-Workflows beginnen mit einem Split Out.

**Fallen:**
- Split Out teilt nur das Feld, das ihr nennt. Steht das Array in `leads` und ihr tragt `data` ein, kommt nichts Brauchbares raus.
- Nur die oberste Ebene wird geteilt. Arrays in Arrays brauchen einen zweiten Split Out.
- Ein leeres Array ergibt null Items. Danach läuft nichts mehr, ohne Fehlermeldung.

### Aggregate – `Playground – Aggregate`

**Bild:** Das Gegenstück zu Split Out. Alle bearbeiteten Briefe kommen zurück in den Umschlag, damit ihr ein Paket verschicken könnt.

**Frage:** Warum will man am Ende oft wieder ein einziges Item?
**Antwort:** Für alles, was *einmal* passieren soll: eine Zusammenfassungs-Mail, ein Webhook-Payload, ein API-Aufruf mit der ganzen Liste. Fünf Items an Google Sheets ergeben dagegen fünf Zeilen, das ist dort richtig.

**Fallen:**
- Zielfeld bewusst benennen (in der Challenge `processed_leads`).
- Nach einem IF erst mergen, dann aggregieren. Sonst sammelt Aggregate nur einen Zweig ein.

### IF – `Playground – IF`

**Bild:** Eine Weiche. Je nach Bedingung fährt der Zug links (true) oder rechts (false) weiter.

**Frage:** Was passiert mit Items im false-Zweig, wenn dort kein Node hängt?
**Antwort:** Sie sind weg. Kein Fehler, keine Warnung. Manchmal gewollt, oft die Antwort auf „Wo sind meine Daten?“

**Fallen:**
- Mehrere Bedingungen sind standardmäßig mit **AND** verknüpft. Wer OR braucht, muss umstellen.
- Zahlen mit Typ **Number** vergleichen. Mit String ist `"100" < "60"`.

### Switch – `Playground – Switch`

**Bild:** Ein Verteilerkasten mit Fächern A, B, C und auf Wunsch einem Fach „Sonstiges“ (Fallback).

**Frage:** Wann nehmt ihr Switch statt mehrerer IF hintereinander?
**Antwort:** Ab drei Fällen. Die ganze Verteillogik steht an einer Stelle, und die Ausgänge lassen sich benennen.

**Fallen:**
- Ohne **Fallback Output** fallen Items, die zu keiner Regel passen, still weg. Im Playground live einschalten.
- Ein Item geht nur in die **erste** passende Regel. Für mehrere Ausgänge *Send data to all matching outputs* aktivieren.
- Regeln werden von oben nach unten geprüft: die speziellen nach oben.

### Code – `Playground – Code statt Split Out`

**Bild:** Die anderen Nodes sind Werkzeuge aus dem Baumarkt. Code ist die Werkstatt, in der ihr euch eins selbst baut. Mehr Freiheit, mehr Verantwortung.

**Frage:** Was kann Code, was Edit Fields nicht kann?
**Antwort:** Über mehrere Items gleichzeitig rechnen, Arrays umbauen, Schleifen und verschachtelte Bedingungen. Für einfaches Setzen und Umbenennen bleibt Edit Fields die bessere Wahl.

**Fallen:**
- Rückgabe ist immer eine Liste: `return [{ json: { … } }]`, auch für ein einziges Item.
- Zwei Modi: *Run Once for All Items* (Standard, `$input.all()`) und *Run Once for Each Item* (`$input.item`). Wer im ersten Modus nur `$input.first()` nimmt, verliert die übrigen Items.
- Ein Fehler im Code stoppt den Workflow. Erst mit *Execute step* testen.

### Merge – `Playground – Merge`

**Bild:** Zwei Züge fahren gleichzeitig los, einer hält unterwegs (`Playground – Wait`). Am Zielbahnhof wartet Merge, bis beide da sind.

Hier der wichtige Satz: **n8n ist kein Skript, das Zeile für Zeile läuft.** Zweige laufen unabhängig, Merge ist der Treffpunkt.

**Frage:** Was ist das Problem, wenn ihr nach einem IF beide Zweige getrennt weiterführt?
**Antwort:** Alles, was danach *alle* Items braucht (Aggregate, eine Mail, ein Sheet-Eintrag), bekommt nur den Zweig, der angeschlossen ist.

**Fallen:**
- Welcher Zweig auf Input 1 und welcher auf Input 2 landet, hängt davon ab, wie die Verbindungen gezogen sind. Bei *Append* egal, bei *Combine* nicht.
- *Combine* verknüpft Items paarweise. Bei ungleicher Anzahl kommt Unerwartetes raus. Hier ist *Append* richtig.

### Edit Fields (Set) – kommt in Aufgabe 3

Kein eigener Playground-Node, aber kurz erwähnen. **Bild:** Ein Formular, in das ihr neue Felder einfügt oder bestehende ändert, ohne den Rest anzufassen. Das klappt nur mit **Include Other Input Fields**. Ohne die Option bleiben nur die neuen Felder übrig. Das ist der häufigste Fehler überhaupt.

Weitere Falle: Ein Tippfehler im Feldnamen legt still ein zweites Feld an (`score` und `Score`).

### Übergang zur Challenge

„Ihr habt die Nodes jetzt einmal live gesehen. In der Challenge baut ihr damit eine echte Pipeline, von rohen Leads bis zum Ergebnis im Scoreboard. Ich starte die Zeit, wenn alle bereit sind.“

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
