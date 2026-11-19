# Modul 0 – Grundlagen: UI, JSON & Data Mapping

| ⏱ Dauer | 30 Minuten (15 min gemeinsam, 15 min Aufgaben) | 5 Nodes | 👥 jede:r für sich | 🏆 Scoreboard |
|---|---|---|---|---|

**Workflow-Datei:** [`modul-00-grundlagen-challenge.json`](modul-00-grundlagen-challenge.json)

**Erklärungen zum Modul** (was der Trainer am Beamer zeigt, zum Nachlesen): [TRAINER.md](TRAINER.md)

> 💡 Handout im **zweiten Browser-Tab** offen lassen. Code-Blöcke haben oben rechts einen Kopieren-Knopf. Voraussetzung: keine.

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

1. **Gemeinsam** (15 min am Beamer): Die Grundlagen aus der [TRAINER.md](TRAINER.md) durchgehen.
2. **Playground** (links im Workflow): `Playground – Bestellung` und `Playground – Expressions` mit **Execute step** ausführen, Input mit Output vergleichen, Expressions ändern.
3. **Aufgaben** (rechts): Die Nodes sind schon verbunden. Ihr tragt in jedem Node die Felder ein, von links nach rechts.
4. Einmal **Execute workflow** klicken, dann jeden Aufgaben-Node mit **Execute step** testen, bevor ihr weitergeht.
5. Der Node **Ergebnis prüfen** zeigt im Feld `hinweise`, was noch nicht stimmt.
6. Tragt in **AUFGABE 0** euren Namen ein. Daraus liest das Scoreboard, wer abgegeben hat.

---

## Aufgaben

In jedem Aufgaben-Node: **Add Field** → Name eintragen → Typ wählen → Wert als Expression. *Include Other Input Fields* ist schon eingeschaltet, damit die Felder der Aufgaben davor erhalten bleiben.

### 00 · SET – Namen eintragen

Der erste Node nach dem Start. Hier steht euer Name fürs Scoreboard.

| Name | Typ | Wert |
|---|---|---|
| name | String | euer Vor- und Nachname, z. B. `Anna Müller` |

Kontrolle: Output zeigt `name` mit eurem Namen. Ist das Feld leer, meldet `Ergebnis prüfen` „AUFGABE 0: Euer Name fehlt“ und es landet nichts im Scoreboard.

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

**Execute workflow** klicken. `Ergebnis prüfen` muss `korrekt: ✅ Ja` zeigen, dann trägt der letzte Node euer Ergebnis ins Scoreboard ein.

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

Die schnellste **korrekte** Abgabe gewinnt. Korrekt heißt: alle sieben Felder mit dem richtigen Wert und dem richtigen Typ. Falsche Abgaben zählen nicht. Details: [Scoreboard](../scoreboard.md).

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
