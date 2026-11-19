# Modul 0 – Grundlagen: UI, JSON & Data Mapping

| ⏱ Dauer | 30 Minuten (15 min gemeinsam, 15 min Aufgaben) | 5 Nodes | 👥 jede:r für sich | 🏆 ohne Scoreboard |
|---|---|---|---|---|

**Workflow-Datei:** [`modul-00-grundlagen-challenge.json`](modul-00-grundlagen-challenge.json)

> 💡 Handout im **zweiten Browser-Tab** offen lassen. Code-Blöcke haben oben rechts einen Kopieren-Knopf. Voraussetzung: keine.

Neu in v02. Im ersten Workshop hat Modul 1 dreimal so lange gedauert wie geplant, weil Oberfläche, JSON und Data Mapping nebenbei gelernt werden mussten. Dieses Modul zieht das vor.

---

## 🎯 Szenario

Ihr bekommt eine Bestellung als JSON. Holt mit **Expressions** einzelne Werte heraus und legt sie als neue Felder an: aus verschachtelten Objekten, aus Arrays, aus einem anderen Node und als Ja/Nein-Wert.

**Input (steckt schon im Node `Beispieldaten`):**

```json
{
  "orderId": "A-1042",
  "Status": "offen",
  "customer": {
    "name": "Miriam Koch",
    "email": "miriam.koch@example.com",
    "address": { "city": "Mannheim", "zip": "68159" }
  },
  "items": [
    { "product": "Laptop-Ständer", "quantity": 2, "price": 39.9 },
    { "product": "USB-C Dock", "quantity": 1, "price": 129 },
    { "product": "Maus", "quantity": 3, "price": 24.5 }
  ],
  "total": 282.3
}
```

## So arbeitet ihr

1. **Gemeinsam** (15 min am Beamer): Die Grundlagen unten durchgehen.
2. **Playground** (links im Workflow): `Playground – Bestellung` und `Playground – Expressions` mit **Execute step** ausführen, Input mit Output vergleichen, Expressions ändern.
3. **Aufgaben** (rechts): Die Nodes sind schon verbunden. Ihr tragt in jedem Node die Felder ein, von links nach rechts.
4. Einmal **Execute workflow** klicken, dann jeden Aufgaben-Node mit **Execute step** testen, bevor ihr weitergeht.
5. Der Node **Ergebnis prüfen** zeigt im Feld `hinweise`, was noch nicht stimmt.

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

---

## Aufgaben

In jedem Aufgaben-Node: **Add Field** → Name eintragen → Typ wählen → Wert als Expression. *Include Other Input Fields* ist schon eingeschaltet, damit die Felder der Aufgaben davor erhalten bleiben.

### 01 · SET – Verschachtelte Felder

Holt Werte aus dem Objekt `customer`. Mit Punkten geht ihr eine Ebene tiefer.

| Name | Typ | Wert |
|---|---|---|
| `kunde` | String | `{{ $json.customer.name }}` |
| `stadt` | String | `{{ $json.customer.address.city }}` |

Kontrolle: `kunde` = `Miriam Koch`, `stadt` = `Mannheim`.

### 02 · SET – Arrays

Holt Werte aus der Liste `items`. Das erste Element hat den Index 0.

| Name | Typ | Wert |
|---|---|---|
| `erster_artikel` | String | `{{ $json.items[0].product }}` |
| `anzahl_positionen` | **Number** | `{{ $json.items.length }}` |

Kontrolle: `erster_artikel` = `Laptop-Ständer`, `anzahl_positionen` = `3`.

### 03 · SET – Groß-/Kleinschreibung

Probiert zuerst `{{ $json.status }}` (kleines s) und schaut, was im Output passiert. Dann korrigieren:

| Name | Typ | Wert |
|---|---|---|
| `status_text` | String | `{{ $json.Status }}` |

⚠️ Im Input heißt das Feld `Status` mit großem S. `$json.status` liefert einfach nichts, ohne Fehlermeldung.

Kontrolle: `status_text` = `offen`.

### 04 · SET – Anderen Node referenzieren

Holt die Bestellnummer nicht aus dem direkten Input, sondern direkt aus dem Node `Beispieldaten`.

| Name | Typ | Wert |
|---|---|---|
| `bestellnummer` | String | `{{ $('Beispieldaten').item.json.orderId }}` |

Der Name in `$('…')` muss exakt dem Node-Namen entsprechen.

Kontrolle: `bestellnummer` = `A-1042`.

### 05 · SET – Ja/Nein-Wert (Boolean)

Ist die Bestellung größer als 100 EUR? Ein Vergleich wie `>` ergibt `true` oder `false`.

| Name | Typ | Wert |
|---|---|---|
| `grosse_bestellung` | **Boolean** | `{{ $json.total > 100 }}` |

⚠️ Typ **Boolean** wählen. Mit Typ String steht im Output der Text `"true"` statt des Werts `true`.

Kontrolle: `grosse_bestellung` = `true`, im Output ohne Anführungszeichen.

### Kontrolle

**Execute workflow** klicken. `Ergebnis prüfen` muss `korrekt: ✅ Ja` zeigen.

### Pinnen ausprobieren

Output von `Beispieldaten` pinnen, dann **Execute workflow**: Der Node läuft nicht mehr neu.

---

## Erwartetes Ergebnis

| Feld | Wert | Typ |
|---|---|---|
| kunde | Miriam Koch | String |
| stadt | Mannheim | String |
| erster_artikel | Laptop-Ständer | String |
| anzahl_positionen | 3 | Number |
| status_text | offen | String |
| bestellnummer | A-1042 | String |
| grosse_bestellung | true | Boolean |

## 🏆 Wertung

In diesem Modul gibt es kein Scoreboard. Hier wird nur geübt. Ab Modul 1 zählt die schnellste **korrekte** Abgabe, siehe [Scoreboard](../scoreboard.md).

## 🔧 Wenn es nicht klappt

| Symptom | Ursache |
|---|---|
| Feld ist leer, aber kein Fehler | Pfad oder Schreibweise falsch, z. B. `status` statt `Status` |
| `[undefined]` oder `null` im Output | Index im Array falsch oder Klammern vergessen: `items[0].product` |
| Felder aus der Aufgabe davor fehlen | *Include Other Input Fields* wurde ausgeschaltet |
| `Ergebnis prüfen` meldet „ist aber Text" | Typ im Set-Node steht auf String statt Number bzw. Boolean |
| `$('Beispieldaten')` meldet einen Fehler | Node-Name falsch geschrieben, exakt `Beispieldaten` |
| Expression wird als Text ausgegeben | Feld steht auf *Fixed* statt *Expression*. Oder die `{{ }}` fehlen |

---

## 🚀 Schon fertig?

1. **Text und Expressions mischen:** Feld `gesamt_text` = `Bestellung A-1042 über 282.3 EUR`.
2. **Teuerster Artikel:** Feld `teuerster_artikel`. Tipp: `{{ $json.items.sort((a, b) => b.price - a.price)[0].product }}`
3. **Zweiter Boolean:** Feld `ist_offen` (Boolean) = `true`, wenn der Status `offen` ist.
4. **Ansichten:** Im Output zwischen **Schema**, **Table** und **JSON** wechseln und die Unterschiede ansehen.
5. [Tastenkürzel](https://docs.n8n.io/keyboard-shortcuts/) ausprobieren: `Tab` (Node suchen), `D` (deaktivieren), `P` (pinnen).
6. Helft dem Nachbarn beim Debuggen.
