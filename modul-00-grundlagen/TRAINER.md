# Modul 0 – Grundlagen: UI, JSON & Data Mapping – Trainer

Ablauf und Erklärungen für den Trainer. Die Aufgaben für die Teilnehmer stehen in der [README.md](README.md).

| Teil | Dauer | Wer |
|---|---|---|
| Demo am Beamer mit dem **Playground** | 15 min | Trainer zeigt, alle schauen zu oder klicken mit |
| Aufgaben 0–5 | 15 min | jede:r für sich |
| Auflösung | 3–5 min | Trainer |

Warum es dieses Modul gibt: Im ersten Workshop hat Modul 1 dreimal so lange gedauert wie geplant, weil Oberfläche, JSON und Data Mapping nebenbei gelernt werden mussten. Dieses Modul zieht das vor.

---

## Vorbereitung

- Vorlage `modul-00-grundlagen-challenge.json` liegt bei allen in n8n (Setup-Skript oder Import aus dem öffentlichen Repo).
- Eigene Kopie am Beamer offen, Browser-Zoom auf 125–150 %.
- Links im Workflow liegt der **Playground** (gelbe Notiz mit zwei Nodes), rechts die Aufgaben. Die Playground-Nodes hängen **nicht** am Trigger. Ist bei jemandem eine Verbindung vom Start zum Playground zu sehen, wurde sie von Hand gezogen: löschen.

---

## Ablauf am Beamer (15 min)

Alle Schritte im **Playground**. Die Teilnehmer dürfen mitklicken, die Aufgaben rechts noch nicht anfangen.

### 1 · Oberfläche (2 min)

1. Canvas zeigen: Playground links, Aufgaben rechts, Daten fließen von links nach rechts.
2. **Playground – Bestellung** doppelklicken: links **Input**, Mitte **Einstellungen**, rechts **Output**.
3. Sagen: „Jeder Node hat diese drei Bereiche. Wer Input und Output vergleicht, versteht, was ein Node macht.“

### 2 · Trigger und Execute step (2 min)

1. Auf **Start – Execute workflow** zeigen (Blitz ⚡): „Hier startet der ganze Workflow.“
2. Sagen: „Der Playground hängt nicht am Trigger. Deshalb führen wir seine Nodes einzeln aus.“
3. In **Playground – Bestellung** auf **Execute step** klicken.

### 3 · JSON lesen (3 min)

1. Output auf **JSON** umschalten. Die Bestellung zeigen:

   | Im Output | Erklären |
   |---|---|
   | `"orderId": "B-2001"` | Feld mit Text (String) |
   | `"customer": { … }` | Objekt: Felder in Feldern |
   | `"address": { "city": "Karlsruhe", "zip": "76131" }` | verschachteltes Objekt |
   | `"items": [ … ]` | Array: Liste, gezählt ab **0** |
   | `"total": 238.8` | Zahl (Number), ohne Anführungszeichen |
   | `"expressVersand": false` | Ja/Nein (Boolean), ohne Anführungszeichen |
   | `"Status"` | **großes S**, merken für später |

2. Auf **Schema** umschalten: „Hier könnt ihr Felder greifen und ziehen.“

### 4 · Expressions (4 min)

1. **Playground – Expressions** öffnen, **Execute step**.
2. Die fünf Felder von oben nach unten durchgehen, jeweils Expression und Ergebnis zeigen:

   | Feld | Expression | Ergebnis | Was man daran sieht |
   |---|---|---|---|
   | `bestellung` | `{{ $json.orderId }}` | `B-2001` | einfaches Feld |
   | `plz` | `{{ $json.customer.address.zip }}` | `76131` | Punkt-Notation für verschachtelte Felder |
   | `menge_erste_position` | `{{ $json.items[0].quantity }}` | `1` | Array-Index beginnt bei 0, Typ **Number** |
   | `express` | `{{ $json.expressVersand }}` | `false` | Typ **Boolean** |
   | `satz` | `Bestellung {{ $json.orderId }} geht nach {{ $json.customer.address.city }}` | `Bestellung B-2001 geht nach Karlsruhe` | Text und Expressions mischen |

3. **Live ändern:** in `plz` die Expression auf `{{ $json.customer.email }}` ändern → Output zeigt die E-Mail.
4. **Drag & Drop:** *Add Field*, Name `kunde`, dann im Input (Schema) `customer.name` greifen und ins Wertfeld ziehen → n8n schreibt `{{ $json.customer.name }}` selbst.
5. **Typ-Falle:** bei `menge_erste_position` den Typ auf **String** stellen → im Output steht `"1"` mit Anführungszeichen. Zurückstellen.

### 5 · Die drei häufigsten Fehler live (2 min)

1. In `bestellung` die Expression auf `{{ $json.status }}` ändern (kleines s) → **leer, aber keine Fehlermeldung**. „Groß-/Kleinschreibung zählt, und n8n warnt euch nicht.“ Zurück auf `orderId`.
2. Node-Namen: „`$('Beispieldaten')` funktioniert nur mit exakt diesem Namen. Kopiert ihr einen Node, heißt er `… 1`.“
3. „Grün heißt nur: kein technischer Fehler. Ob das Ergebnis stimmt, seht ihr im Output. Deshalb gibt es in jeder Aufgabe den Node **Ergebnis prüfen**.“

### 6 · Pinnen (1 min)

1. In **Playground – Bestellung** im Output auf die Stecknadel 📌 klicken.
2. **Playground – Expressions** erneut ausführen: Die Bestellung wird nicht neu erzeugt, die gepinnten Daten werden verwendet.
3. „Spart Zeit und API-Aufrufe, wenn ein Node langsam ist oder etwas kostet.“ Stecknadel wieder lösen.

### 7 · Übergang zu den Aufgaben (1 min)

1. README im **zweiten Browser-Tab** öffnen lassen.
2. **AUFGABE 0:** Namen eintragen (bei Kopien aus dem Setup-Skript steht er schon drin).
3. Aufgaben 1–5 von links nach rechts, nach jeder Aufgabe **Execute step** und Output anschauen.
4. Am Ende **Execute workflow**. `Ergebnis prüfen` sagt im Feld `hinweise`, was noch fehlt.
5. Scoreboard-Regel nennen: **erst richtig, dann schnell.** Die Rangliste am Beamer zeigen.

---

## Während der Übung

- Rundgang. Die häufigsten Probleme und ihre Ursachen stehen in der README unter „Wenn es nicht klappt“.
- Wer fertig ist: Bonusaufgaben aus „Schon fertig?“, danach Nachbarn helfen.

## Auflösung (3–5 min)

1. Lösungs-Workflow am Beamer öffnen, Aufgaben 1–5 kurz durchklicken.
2. Die typische Falle des Moduls besprechen: Aufgabe 3 (`Status` mit großem S).
3. Rangliste zeigen.

---

# Zum Nachschlagen

## Die Oberfläche in 5 Minuten

| Bereich | Wofür |
|---|---|
| **Canvas** | Die Arbeitsfläche. Nodes werden von links nach rechts verbunden. |
| **Node öffnen** | Doppelklick. Links der **Input**, in der Mitte die **Einstellungen**, rechts der **Output**. |
| **Execute workflow** | Startet beim Trigger und führt alles aus. |
| **Execute step** | Führt nur den geöffneten Node aus (und was davor noch fehlt). |
| **Executions** (Reiter oben) | Alle bisherigen Läufe zum Nachschauen. |

Ansicht im Input und Output umschalten: **Schema** (zum Ziehen von Feldern), **Table**, **JSON**.

## Trigger: Wie startet ein Workflow?

Jeder Workflow braucht einen **Trigger**. Das ist immer der erste Node, erkennbar am Blitz ⚡.

- In den Modulen 0 bis 3 ist das der **Manual Trigger**: Er startet, wenn ihr auf **Execute workflow** klickt.
- Andere Trigger (Zeitplan, Formular, Webhook, E-Mail, Chat) lernt ihr in Modul 4.
- Ohne Trigger kann ein Workflow nicht als Ganzes laufen. Einzelne Nodes lassen sich trotzdem mit **Execute step** testen. So funktioniert auch der Playground.

## JSON lesen

Alle Daten in n8n sind **Items**. Ein Item ist ein JSON-Objekt wie die Bestellung im Playground.

| Zeichen | Bedeutung | Beispiel |
|---|---|---|
| `{ }` | Objekt: Felder mit Namen | `customer` |
| `[ ]` | Array: Liste, gezählt ab 0 | `items` |
| `"text"` | String (Text) | `"offen"` |
| `282.3` | Number (ohne Anführungszeichen) | `total` |
| `true` / `false` | Boolean (Ja/Nein, ohne Anführungszeichen) | Ergebnis von `total > 100` |

## Data Mapping mit Expressions

Eine **Expression** steht in doppelten geschweiften Klammern und holt einen Wert aus den Daten.

| Ich will … | Expression |
|---|---|
| ein Feld | `{{ $json.orderId }}` |
| ein verschachteltes Feld | `{{ $json.customer.address.zip }}` |
| ein Feld aus einem Array-Element | `{{ $json.items[1].price }}` |
| einen Vergleich (ergibt true/false) | `{{ $json.Status === 'offen' }}` |
| einen Wert aus einem früheren Node | `{{ $('Playground – Bestellung').item.json.total }}` |

Der einfachste Weg: Feld links im **Input (Schema)** greifen und in das Eingabefeld ziehen. n8n schreibt die Expression selbst.

## Arrays in Expressions

Die wichtigsten Expressions für Listen, mit den Ergebnissen für die Bestellung aus `Beispieldaten` (siehe README, Szenario):

| Ich will … | Expression | Ergebnis |
|---|---|---|
| das erste Element | `{{ $json.items[0].product }}` | `Laptop-Ständer` |
| das letzte Element | `{{ $json.items.last().product }}` | `Maus` |
| die Anzahl | `{{ $json.items.length }}` | `3` |
| ein Feld aus allen Elementen | `{{ $json.items.pluck('product') }}` | `["Laptop-Ständer", "USB-C Dock", "Maus"]` |
| … als Text | `{{ $json.items.pluck('product').join(', ') }}` | `Laptop-Ständer, USB-C Dock, Maus` |
| die Summe | `{{ $json.items.pluck('quantity').sum() }}` | `6` |
| den höchsten Wert | `{{ $json.items.pluck('price').max() }}` | `129` |
| nur passende Elemente | `{{ $json.items.filter(i => i.price > 30).length }}` | `2` |
| ein bestimmtes Element | `{{ $json.items.find(i => i.product === 'Maus').price }}` | `24.5` |
| prüfen, ob etwas enthalten ist | `{{ $json.items.pluck('product').includes('Maus') }}` | `true` |
| prüfen, ob die Liste leer ist | `{{ $json.items.isEmpty() }}` | `false` |

> 💡 Diese Expressions holen Werte aus einem Array **innerhalb eines Items**. Soll jedes Element als eigenes Item weiterlaufen (z. B. jede Position einzeln prüfen), braucht ihr den Node **Split Out**. Den lernt ihr in Modul 1.

## ⚠️ Die drei häufigsten Fehler

1. **Groß-/Kleinschreibung zählt.** `$json.Status` und `$json.status` sind zwei verschiedene Felder. Das falsche liefert einfach einen leeren Wert, **ohne Fehlermeldung**.
2. **Node-Namen müssen exakt stimmen.** `$('Beispieldaten')` funktioniert nur, wenn der Node genau so heißt. Kopiert ihr einen Node, hängt n8n eine `1` an den Namen.
3. **Der Workflow läuft grün durch und das Ergebnis ist trotzdem falsch.** Grün heißt nur: kein technischer Fehler. Schaut euch immer den Output an.

## Daten pinnen 📌

Im Output eines Nodes oben rechts auf die **Stecknadel** klicken. Der Output ist dann eingefroren:

- Der Node wird nicht neu ausgeführt, die Nodes dahinter arbeiten mit den gepinnten Daten.
- Spart Zeit und API-Aufrufe beim Bauen.
- Zum Lösen noch einmal auf die Stecknadel klicken.
