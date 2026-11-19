# Modul 0 – Grundlagen: UI, JSON & Data Mapping – Erklärungen

Was der Trainer am Beamer erklärt. Die Aufgaben stehen in der [README.md](README.md).

---

Neu in v02. Im ersten Workshop hat Modul 1 dreimal so lange gedauert wie geplant, weil Oberfläche, JSON und Data Mapping nebenbei gelernt werden mussten. Dieses Modul zieht das vor.

---

## Grundlagen

### Die Oberfläche in 5 Minuten

| Bereich | Wofür |
|---|---|
| **Canvas** | Die Arbeitsfläche. Nodes werden von links nach rechts verbunden. |
| **Node öffnen** | Doppelklick. Links der **Input**, in der Mitte die **Einstellungen**, rechts der **Output**. |
| **Execute workflow** | Startet beim Trigger und führt alles aus. |
| **Execute step** | Führt nur den geöffneten Node aus (und was davor noch fehlt). |
| **Executions** (Reiter oben) | Alle bisherigen Läufe zum Nachschauen. |

Ansicht im Input und Output umschalten: **Schema** (zum Ziehen von Feldern), **Table**, **JSON**.

### Trigger: Wie startet ein Workflow?

Jeder Workflow braucht einen **Trigger**. Das ist immer der erste Node, erkennbar am Blitz ⚡.

- In den Modulen 0 bis 3 ist das der **Manual Trigger**: Er startet, wenn ihr auf **Execute workflow** klickt.
- Andere Trigger (Zeitplan, Formular, Webhook, E-Mail, Chat) lernt ihr in Modul 4.
- Ohne Trigger kann ein Workflow nicht als Ganzes laufen. Einzelne Nodes lassen sich trotzdem mit **Execute step** testen. So funktioniert auch der Playground.

### JSON lesen

Alle Daten in n8n sind **Items**. Ein Item ist ein JSON-Objekt wie die Bestellung oben.

| Zeichen | Bedeutung | Beispiel |
|---|---|---|
| `{ }` | Objekt: Felder mit Namen | `customer` |
| `[ ]` | Array: Liste, gezählt ab 0 | `items` |
| `"text"` | String (Text) | `"offen"` |
| `282.3` | Number (ohne Anführungszeichen) | `total` |
| `true` / `false` | Boolean (Ja/Nein, ohne Anführungszeichen) | Ergebnis von `total > 100` |

### Data Mapping mit Expressions

Eine **Expression** steht in doppelten geschweiften Klammern und holt einen Wert aus den Daten.

| Ich will … | Expression |
|---|---|
| ein Feld | `{{ $json.orderId }}` |
| ein verschachteltes Feld | `{{ $json.customer.address.zip }}` |
| ein Feld aus einem Array-Element | `{{ $json.items[1].price }}` |
| einen Vergleich (ergibt true/false) | `{{ $json.Status === 'offen' }}` |
| einen Wert aus einem früheren Node | `{{ $('Playground – Bestellung').item.json.total }}` |

Der einfachste Weg: Feld links im **Input (Schema)** greifen und in das Eingabefeld ziehen. n8n schreibt die Expression selbst.

### Arrays in Expressions

Die wichtigsten Expressions für Listen, mit den Ergebnissen für die Bestellung oben:

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

### ⚠️ Die drei häufigsten Fehler

1. **Groß-/Kleinschreibung zählt.** `$json.Status` und `$json.status` sind zwei verschiedene Felder. Das falsche liefert einfach einen leeren Wert, **ohne Fehlermeldung**.
2. **Node-Namen müssen exakt stimmen.** `$('Beispieldaten')` funktioniert nur, wenn der Node genau so heißt. Kopiert ihr einen Node, hängt n8n eine `1` an den Namen.
3. **Der Workflow läuft grün durch und das Ergebnis ist trotzdem falsch.** Grün heißt nur: kein technischer Fehler. Schaut euch immer den Output an.

### Daten pinnen 📌

Im Output eines Nodes oben rechts auf die **Stecknadel** klicken. Der Output ist dann eingefroren:

- Der Node wird nicht neu ausgeführt, die Nodes dahinter arbeiten mit den gepinnten Daten.
- Spart Zeit und API-Aufrufe beim Bauen.
- Zum Lösen noch einmal auf die Stecknadel klicken.
