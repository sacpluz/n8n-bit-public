# Modul 3 – Workflows steuern: Angebotsvergleich – Erklärungen

Was der Trainer am Beamer erklärt. Die Aufgaben stehen in der [README.md](README.md).

---

## Ablauf am Beamer (10 min) – Playground links im Workflow

Alle Nodes in der gelben Notiz **Playground – Loop und HTTP Request**.

| Schritt | Node | Was zeigen |
|---|---|---|
| 1 | **Playground – HTTP Request (USD → EUR)** | *Execute step*: Antwort der Wechselkurs-API, Feld `rates.EUR`. Genau diese API nutzt Aufgabe 2 |
| 2 | **Playground – Drei Angebote** | 3 Items |
| 3 | **Playground – Fertig (done)** | *Execute step* startet den Loop. Ergebnis: alle 3 Items, jedes mit `durchlauf` |
| 4 | **Playground – Pro Durchlauf** | Im Output oben zwischen Run 1, 2, 3 umschalten: pro Durchlauf ein Item |
| 5 | **Playground – Loop Over Items** | Die zwei Ausgänge zeigen: **loop** (zurück in die Schleife) und **done** (am Ende) |

Danach in den Aufgaben auf **Loop Over Items** zeigen: gleiches Muster, nur mit HTTP Request und Code in der Schleife.

## So funktioniert der Loop

Der Node **Loop Over Items** ist schon verdrahtet und hat zwei Ausgänge:

- **loop**: gibt ein Angebot nach dem anderen in die Schleife (HTTP Request → Code → zurück in den Loop).
- **done**: gibt am Ende alle gesammelten Ergebnisse weiter.
