# Scoreboard – so funktioniert der Wettbewerb

Jeder Challenge-Workflow schickt sein Ergebnis am Ende an ein gemeinsames **Google Sheet**. Ein kleines Apps Script im Sheet nimmt die Abgabe an, die Rangliste rechnet das Sheet selbst aus. Niemand braucht dafür eine Credential.

## Die Regel

**Die Logik entscheidet, nicht das Tempo allein.**

1. Nur **korrekte** Abgaben kommen in die Wertung.
2. Unter den korrekten gewinnt die **früheste**.
3. Pro Name zählt die **früheste korrekte** Abgabe. Mehrfach abgeben schadet nicht und bringt keine zusätzlichen Plätze.
4. Modul 4: Es gewinnt, wer zuerst **alle fünf Trigger** (Formular, Webhook, Schedule, Chat, Gmail) ausgelöst hat.

Ein schneller, falscher Workflow bringt nichts. Erst richtig, dann schnell.

## Was technisch passiert

```
Teilnehmer-Workflow
… → Ergebnis prüfen → Scoreboard – Ergebnis eintragen ──POST──▶ Apps Script ──▶ Tab „Abgaben" ──▶ Tab „Rangliste" (Formeln)
```

| Schritt | Was |
|---|---|
| **Ergebnis prüfen** | Prüft das Ergebnis, setzt `korrekt` und nennt im Feld `hinweise`, was noch nicht stimmt |
| **Scoreboard – Ergebnis eintragen** | HTTP Request an das Apps Script im Sheet: Name, Modul, Korrekt, Details |
| **Apps Script** | Setzt die Zeit und hängt eine Zeile an `Abgaben` an |
| **Rangliste** | Tab im Sheet, rechnet aus den Abgaben die Plätze aus |

- Der **Name** kommt aus dem Workflow-Namen: alles vor dem ersten ` – `. Deshalb den Workflow nicht umbenennen.
- Die **Zeit** setzt das Apps Script beim Eintragen, also für alle auf derselben Uhr.
- In Modul 4 schreibt jeder Trigger eine eigene Zeile (`Scoreboard – Formular eintragen` usw.).

## Was gewertet wird

| Modul | Korrekt, wenn |
|---|---|
| 0 | alle sieben Felder den richtigen Wert haben, `anzahl_positionen` als Zahl und `grosse_bestellung` als Boolean |
| 1 | genau die drei richtigen Leads `Qualifiziert` / `Hoch` und die zwei anderen `Abgelehnt` / `Niedrig` sind |
| 2 | genau die fünf erwarteten Abonnenten in der richtigen Reihenfolge ankommen |
| 3 | ein bekannter Lieferant und ein Preis größer 0 ankommen. Der Kurs ändert sich täglich, deshalb wird der Betrag nicht fest geprüft |
| 4 | jeder der fünf Trigger einmal ausgelöst hat |

Ob eine Abgabe korrekt ist, entscheidet der Node `Ergebnis prüfen` im Workflow. Er steht als „BEREITS FERTIG – nicht verändern" im Workflow.
