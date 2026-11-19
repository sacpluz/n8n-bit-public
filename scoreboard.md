# Scoreboard – so funktioniert der Wettbewerb

Ein zentraler n8n-Workflow des Facilitators nimmt die Abgaben entgegen, prüft sie und zeigt die Rangliste als Webseite. Teilnehmer brauchen dafür keine Credential.

## Die Regel

**Die Logik entscheidet, nicht das Tempo allein.**

1. Nur **korrekte** Abgaben kommen in die Wertung.
2. Unter den korrekten gewinnt die **früheste**.
3. Pro Name zählt die **früheste korrekte** Abgabe. Mehrfach abgeben schadet nicht und bringt keine zusätzlichen Plätze.
4. Modul 4: Es gewinnt, wer zuerst **alle fünf Trigger** (Formular, Webhook, Schedule, Chat, Gmail) ausgelöst hat.

Ein schneller, falscher Workflow bringt nichts. Erst richtig, dann schnell.

## Was technisch passiert

```
Teilnehmer-Workflow                         Facilitator-Workflow „Scoreboard"
… → Ergebnis prüfen → Scoreboard –   POST   Webhook – Abgabe → Abgabe prüfen → Data Table
                      Ergebnis senden ────▶
                                            Webhook – Scoreboard anzeigen → Rangliste (HTML)
```

| Schritt | Wo | Was |
|---|---|---|
| **Ergebnis prüfen** | Teilnehmer | Fasst das Ergebnis zusammen und nennt im Feld `hinweise`, was noch nicht stimmt |
| **Scoreboard – Ergebnis senden** | Teilnehmer | HTTP Request an den Scoreboard-Workflow. Schickt Name, Modul und Ergebnis |
| **Abgabe prüfen** | Facilitator | Prüft die Korrektheit noch einmal zentral und setzt den Zeitstempel |
| **Rangliste** | Facilitator | Webseite unter `https://<instanz>/webhook/scoreboard` |

- Der **Name** kommt aus dem Workflow-Namen: alles vor dem ersten ` – `. Deshalb den Workflow nicht umbenennen.
- Der **Zeitstempel** entsteht im Scoreboard-Workflow, also für alle auf derselben Uhr.
- Die **Wertungslogik** steht nur im Scoreboard-Workflow. Sie lässt sich dort nachlesen, aber nicht von Teilnehmern verstellen.
- Die Antwort des HTTP Request zeigt sofort `korrekt: true/false` und den Link zur Rangliste.

## Was gewertet wird

| Modul | Korrekt, wenn |
|---|---|
| 0 | alle sieben Felder den richtigen Wert haben, `anzahl_positionen` als Zahl und `grosse_bestellung` als Boolean |
| 1 | genau die drei richtigen Leads `Qualifiziert` / `Hoch` und die zwei anderen `Abgelehnt` / `Niedrig` sind |
| 2 | genau die fünf erwarteten Abonnenten in der richtigen Reihenfolge ankommen |
| 3 | ein bekannter Lieferant und ein Preis größer 0 ankommen. Der Kurs ändert sich täglich, deshalb wird der Betrag nicht fest geprüft |
| 4 | die Abgabe eine der fünf Trigger-Quellen nennt. Gewertet wird, wer alle fünf hat |
