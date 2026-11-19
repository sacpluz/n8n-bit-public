# Modul 3 – Workflows steuern: Angebotsvergleich

| ⏱ Dauer | 25 Minuten | 5 Aufgaben | 👥 Teams à 2 Personen |
|---|---|---|---|

**Workflow-Datei:** [`modul-03-workflows-steuern-challenge.json`](modul-03-workflows-steuern-challenge.json)

**Erklärungen zum Modul** (was der Trainer am Beamer zeigt, zum Nachlesen): [TRAINER.md](TRAINER.md)

> 💡 Handout im **zweiten Browser-Tab** offen lassen. In diesem Modul kopiert ihr Code von hier in den Workflow.

**Neu in v02:** Start per **Execute workflow** statt per Webhook. Der Webhook ist jetzt eine Bonusaufgabe und wird in Modul 4 richtig erklärt.

---

## 🎯 Szenario

Euer Einkauf hat Angebote von fünf internationalen Lieferanten in fünf Währungen. Ihr baut einen Workflow, der jedes Angebot in einer **Schleife** über eine öffentliche Wechselkurs-API in Euro umrechnet und daraus eine Entscheidungsvorlage macht: günstigstes Angebot, teuerstes Angebot, Durchschnitt.

**Input (steckt schon im Node `Angebote laden`):**

```json
{
  "offers": [
    { "supplier": "TechSupply US",   "price": 1200,   "currency": "USD" },
    { "supplier": "Nordic Parts",    "price": 11000,  "currency": "SEK" },
    { "supplier": "UK Components",   "price": 950,    "currency": "GBP" },
    { "supplier": "Swiss Precision", "price": 1100,   "currency": "CHF" },
    { "supplier": "Japan Direct",    "price": 180000, "currency": "JPY" }
  ]
}
```

⚠️ **Node-Namen nicht ändern.** Der Code in Aufgabe 3 greift mit `$('Loop Over Items')` auf den Loop zu. Der Name muss exakt stimmen, auch Groß-/Kleinschreibung. Kopiert ihr den Node, heißt die Kopie `Loop Over Items1` und der Verweis läuft ins Leere.

---

## Aufgaben

### 00 · SET – Namen eintragen

Der erste Node nach dem Start. Hier steht euer Name fürs Scoreboard.

| Name | Typ | Wert |
|---|---|---|
| name | String | euer Vor- und Nachname, z. B. `Anna Müller` |

Kontrolle: Output zeigt `name` mit eurem Namen. Ist das Feld leer, meldet `Ergebnis prüfen` „AUFGABE 0: Euer Name fehlt“ und es landet nichts im Scoreboard.

### 01 · SPLIT OUT – Angebote aufteilen

Kennt ihr aus Modul 1: aus einem Item mit Array werden fünf Items.

| Einstellung | Wert |
|---|---|
| Fields To Split Out | `offers` |

### 02 · HTTP REQUEST – Wechselkurs abrufen

| Einstellung | Wert |
|---|---|
| Method | **GET** |
| URL | als Expression, siehe unten |

```
https://api.frankfurter.dev/v1/latest?base={{ $json.currency }}&symbols=EUR&amount={{ $json.price }}
```

Die API ist kostenlos und braucht keinen Key. Antwort für das erste Angebot:

```json
{ "amount": 1200, "base": "USD", "date": "2026-11-11", "rates": { "EUR": 1078.46 } }
```

💡 **Pinnen:** Wenn der Request einmal funktioniert hat, könnt ihr den Output pinnen 📌 und sparen euch beim Weiterbauen die API-Aufrufe. Vor der Abgabe wieder lösen.

### 03 · CODE – Lieferant und Kurs zusammenführen

Die API-Antwort kennt den Lieferanten nicht. Dieser Code holt ihn aus dem Loop dazu. Im Code-Node **alles ersetzen** durch:

```javascript
const rate = $input.first().json;
const offer = $('Loop Over Items').item.json;

return [{
  json: {
    supplier: offer.supplier,
    amount: rate.amount,
    base: rate.base,
    rates: rate.rates
  }
}];
```

### 04 · CODE – Entscheidungsvorlage

Im Code-Node **alles ersetzen** durch:

```javascript
const items = $input.all();

const offers = items.map(item => ({
  supplier: item.json.supplier || 'Unknown',
  originalPrice: item.json.amount || 0,
  currency: item.json.base || '',
  priceEUR: item.json.rates?.EUR || 0
}));

offers.sort((a, b) => a.priceEUR - b.priceEUR);

const prices = offers.map(a => a.priceEUR);
const total = prices.reduce((s, p) => s + p, 0);
const average = prices.length > 0 ? Math.round((total / prices.length) * 100) / 100 : 0;

return [{
  json: {
    totalOffers: offers.length,
    cheapestOffer: offers.length > 0 ? { supplier: offers[0].supplier, priceEUR: offers[0].priceEUR } : null,
    mostExpensiveOffer: offers.length > 0 ? { supplier: offers[offers.length - 1].supplier, priceEUR: offers[offers.length - 1].priceEUR } : null,
    averagePriceEUR: average,
    offers: offers
  }
}];
```

Kontrolle: Output zeigt **1 item** mit `totalOffers: 5` und einem Objekt `cheapestOffer`.

### 05 · EDIT FIELDS – Ergebnis fürs Scoreboard mappen

Drei Felder anlegen. Die Werte liegen verschachtelt in `cheapestOffer` (siehe Modul 0).

| Name | Typ | Wert |
|---|---|---|
| `cheapest_offer_supplier` | String | `{{ $json.cheapestOffer.supplier }}` |
| `cheapest_offer_price_EUR` | Number | `{{ $json.cheapestOffer.priceEUR }}` |
| `timestamp` | String | `{{ $now.format('dd.M.yyyy, HH:mm:ss') }}` |

⚠️ Feldnamen exakt so schreiben. `EUR` in Großbuchstaben, der Rest klein.

### Abgabe

**Execute workflow**. `Ergebnis prüfen` bricht mit einer klaren Meldung ab, wenn ein Feld fehlt. Läuft alles durch, steht euer Ergebnis im Scoreboard.

---

## 🏆 Wertung

Die schnellste **korrekte** Abgabe gewinnt. Details: [Scoreboard](../scoreboard.md).

## 🔧 Wenn es nicht klappt

| Symptom | Ursache |
|---|---|
| `Referenced node doesn't exist` | Der Name in `$('Loop Over Items')` passt nicht zum Node-Namen |
| `priceEUR` ist überall 0 | Aufgabe 3 gibt `rates` nicht weiter, oder der HTTP Request liefert einen Fehler |
| HTTP Request: 404 oder leere Antwort | Tippfehler in der URL, oder `currency` / `price` falsch geschrieben |
| Loop läuft nur einmal | Aufgabe 1 fehlt: Es kommt nur ein Item mit Array an |
| Felder in Aufgabe 5 sind leer | Pfad falsch. Richtig ist `$json.cheapestOffer.supplier` (großes O) |

---

## 🚀 Schon fertig?

1. **Webhook-Start (wie in v01):** Ersetzt den Manual Trigger durch einen Webhook (POST, eigener Pfad) und schickt die Angebote als JSON-Body an die **Test URL**. Dann liegt das Array unter `$json.body.offers`. Am Ende einen *Respond to Webhook*-Node anhängen.
2. **Ohne Loop:** Der HTTP Request läuft in n8n ohnehin einmal pro Item. Baut die Schleife aus. Wie kommt ihr jetzt an den Lieferanten? Wann braucht man den Loop wirklich?
3. **Sechstes Angebot:** Ergänzt im Node `Angebote laden` einen Lieferanten in `PLN`.
4. **Bonus-Modul:** [Modul 3.1 – Looping](../modul-03-1-looping-bonus/README.md).
