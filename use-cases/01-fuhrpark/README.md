# Use Case 01 – Fuhrpark: Leasing-Angebote automatisch erfassen

## Szenario

Die Fuhrparkabteilung erhält Leasing-Angebote per E-Mail als PDF-Anhang. Jedes Angebot bezieht sich auf einen Fahrer und ein Fahrzeugmodell. Ein Fahrer kann für dasselbe Modell Angebote von bis zu drei Anbietern einholen: **Alphabet**, **ARVAL** und **VW Leasing**.

Eure Aufgabe: Baut einen Workflow, der eingehende Angebote automatisch auswertet und eine zentrale Übersichtstabelle in Google Sheets pflegt.

https://bitglobal.sharepoint.com/sites/Fuhrpark/SitePages/Beipsielrecher.aspx?csf=1&web=1&e=vIgceB

<img width="1093" height="648" alt="image" src="https://github.com/user-attachments/assets/ea26bdc1-76e2-452d-9edb-976a0da5eae0" />


---

## Gesamtprozess (Ziel)

```
E-Mail empfangen (Trigger)
        ↓
PDF-Anhang herunterladen
        ↓
Daten aus PDF auslesen
        ↓
Google Sheets: Zeile für Fahrer + Modell suchen
        ↓
Fall 1: Neu → Zeile anlegen
Fall 2: Bekannt, Anbieter neu, Slot frei → Slot füllen
Fall 3: Bekannt, Anbieter vorhanden → Werte aktualisieren
Fall 4: Bekannt, alle Slots voll, Anbieter neu → Ältesten überschreiben
```

---

## Daten die aus dem PDF ausgelesen werden

| Feld | Beispielwert |
|---|---|
| Angebotsdatum | 19.11.2025 |
| Anbieter | VW Leasing / Alphabet / ARVAL |
| Fahrer | Doe, John |
| Hersteller | BMW |
| Modell | iX3 50 xDrive |
| Antrieb | Elektro |
| Laufzeit | 36 |
| km p.a. | 25.000 |
| Toleranz km | 2.500 |
| Listenpreis netto | 63.159,66 |
| Gesamtpreis brutto | 75.160,00 |
| Finanzfaktor | 1,0813 |
| Finanzrate netto | 682,96 |
| Reifenservice | 116,19 |
| Full-Service-Rate | 147,05 |
| Gesamtrate brutto | 986,55 |

---

## Google Sheets – Tabellenstruktur

### Fixe Felder *(einmalig beim ersten Angebot gesetzt)*

| Spalte | Beschreibung |
|---|---|
| Fahrer | Nachname, Vorname |
| Hersteller | z. B. BMW |
| Modell | z. B. iX3 50 xDrive |
| Antrieb | Elektro / Diesel / Benzin |
| Laufzeit | in Monaten |
| km p.a. | Laufleistung pro Jahr |
| Toleranz km | Toleranzkilometer |
| Listenpreis netto | in EUR |
| Gesamtpreis brutto | in EUR |
| Finanzrate netto | monatlich in EUR |
| Reifenservice | monatlich in EUR |
| Full-Service-Rate | monatlich in EUR |
| Gesamtrate brutto | monatlich in EUR |

### Variable Felder *(pro Anbieter ergänzt oder aktualisiert)*

| Spalte | Anbieter |
|---|---|
| Anbieter 1 / Finanzfaktor 1 / Datum 1 | Alphabet |
| Anbieter 2 / Finanzfaktor 2 / Datum 2 | ARVAL |
| Anbieter 3 / Finanzfaktor 3 / Datum 3 | VW Leasing |

---

## Stufe 1 – Daten aus dem PDF sehen 🟢 Pflicht

**Ziel:** Den Inhalt eines Leasing-PDFs automatisch auslesen und als strukturierte Daten im Workflow sichtbar machen.

**Vereinfachung:** Kein Mail-Trigger, kein Attachment-Download. Das PDF wird direkt per URL in den Workflow geladen.

### Testdaten

Es stehen **34 Dummy-PDFs** zur Verfügung. Die Dateien sind durchnummeriert und folgen diesem URL-Muster:

```
https://raw.githubusercontent.com/sacpluz/n8n-bit-public/main/fuhrpark/use_case_01_dummy_daten-01.pdf
https://raw.githubusercontent.com/sacpluz/n8n-bit-public/main/fuhrpark/use_case_01_dummy_daten-02.pdf
...
https://raw.githubusercontent.com/sacpluz/n8n-bit-public/main/fuhrpark/use_case_01_dummy_daten-34.pdf
```

Jedes Team wählt eine eigene Nummer — damit testet ihr mit unterschiedlichen Angeboten.

### Aufgaben

**01 · Manueller Trigger**
Startet den Workflow manuell über „Execute Workflow".

**02 · PDF laden via HTTP Request**
Ladet das Leasing-PDF direkt aus GitHub in den Workflow.

- **Method:** `GET`
- **URL:** URL aus der Liste oben (z. B. `-01.pdf`)
- **Response Format:** `File`

> Das Binary landet automatisch im nächsten Node — kein weiterer Download-Schritt nötig.

**03 · Daten auslesen**
Nutzt einen AI Node um die relevanten Felder aus dem PDF zu extrahieren. Gebt dem Node eine klare Liste der Felder die ihr braucht.

**04 · Felder strukturieren**
Nutzt einen Set Node um die extrahierten Werte in sauber benannte Felder zu überführen.

### Erwartetes Ergebnis

Im Output des letzten Nodes sind folgende Felder sichtbar:

```
fahrer, hersteller, modell, antrieb, anbieter, angebotsdatum,
laufzeit, km_pa, toleranz_km, listenpreis_netto, gesamtpreis_brutto,
finanzfaktor, finanzrate_netto, reifenservice, full_service_rate, gesamtrate_brutto
```

---

## Stufe 2 – Einfache Zeile in Google Sheets schreiben 🟢 Pflicht

**Ziel:** Die ausgelesenen Daten in eine neue Zeile in Google Sheets schreiben — ohne Prüfung ob die Zeile schon existiert.

**Vereinfachung:** Kein Lookup, keine Logik. Jedes Angebot erzeugt immer eine neue Zeile. Der Anbieter wird immer in Slot 1 geschrieben.

### Aufgaben

**05 · Google Sheets Node konfigurieren**
Verbindet euren Google Sheets Account. Wählt das Sheet „Leasing-Angebote" und die Operation `append`.

**06 · Felder mappen**
Mappt alle fixen Felder sowie Anbieter 1 / Finanzfaktor 1 / Datum 1 auf die entsprechenden Spalten.

**07 · Testen**
Führt den Workflow mit drei verschiedenen PDFs aus (einfach die Nummer in der URL ändern). Prüft ob alle drei als neue Zeilen erscheinen.

### Erwartetes Ergebnis

| Fahrer | Modell | ... | Anbieter 1 | FF 1 | Datum 1 |
|---|---|---|---|---|---|
| Doe, John | iX3 50 xDrive | ... | Alphabet | 1,0813% | 19.11.2025 |
| Müller, Thomas | ID.4 Pro | ... | Alphabet | 0,8750% | 05.12.2025 |
| Doe, John | iX3 50 xDrive | ... | ARVAL | 1,0240% | 03.12.2025 |

> Die dritte Zeile ist ein Duplikat — das beheben wir in Stufe 3.

---

## Stufe 3 – Duplikate erkennen und aktualisieren 🟢 Pflicht

**Ziel:** Vor dem Schreiben prüfen ob für diesen Fahrer und dieses Modell bereits eine Zeile existiert. Wenn ja, wird die Zeile aktualisiert statt eine neue angelegt.

**Vereinfachung:** Noch keine Slot-Logik. Wenn die Zeile existiert, wird Anbieter 1 / Finanzfaktor 1 / Datum 1 einfach überschrieben.

### Aufgaben

**08 · Lookup konfigurieren**
Nutzt den Google Sheets Node mit der Operation `lookup`. Sucht nach der Kombination `Fahrer + Modell`.

**09 · IF Node: Zeile gefunden?**
Prüft ob der Lookup ein Ergebnis zurückgegeben hat.
- True → Zeile existiert → Update
- False → Zeile neu → Append

**10 · Update konfigurieren**
Nutzt den Google Sheets Node mit der Operation `update`. Aktualisiert Anbieter 1 / Finanzfaktor 1 / Datum 1 anhand der Zeilennummer aus dem Lookup.

**11 · Testen**
Führt den Workflow zuerst mit einem neuen Angebot aus (neue Zeile), dann nochmal mit demselben Fahrer und Modell aber anderem Anbieter (Update).

### Erwartetes Ergebnis

| Fahrer | Modell | ... | Anbieter 1 | FF 1 | Datum 1 |
|---|---|---|---|---|---|
| Doe, John | iX3 50 xDrive | ... | ARVAL | 1,0240% | 03.12.2025 |
| Müller, Thomas | ID.4 Pro | ... | Alphabet | 0,8750% | 05.12.2025 |

> Doe erscheint nur einmal. Das zweite Angebot hat Slot 1 überschrieben.

---

## Stufe 4 – Slot-Logik implementieren 🟡 Bonus

**Ziel:** Anbieter werden in den richtigen Slot geschrieben. Alle vier Fälle werden korrekt behandelt.

### Entscheidungslogik

```
Zeile existiert → Anbieter aus PDF mit Anbieter 1 / 2 / 3 vergleichen

├─ Anbieter bereits in Slot 1, 2 oder 3?
│       → Finanzfaktor + Datum in diesem Slot aktualisieren
│
├─ Anbieter noch nicht vorhanden, freier Slot verfügbar?
│       → Nächsten freien Slot befüllen
│
└─ Alle Slots belegt, Anbieter neu?
        → Datum 1 / 2 / 3 vergleichen → ältesten Slot überschreiben
```

### Aufgaben

**12 · Code Node: Slot-Logik**
Schreibt einen Code Node der die drei Anbieter-Spalten prüft und entscheidet welche Aktion und welcher Slot verwendet wird. Ergebnis: ein Feld `aktion` (`update_slot_1`, `update_slot_2`, `update_slot_3`, `append`) sowie die befüllten Slot-Felder.

**13 · Switch Node: Aktion auswählen**
Leitet den Datenstrom anhand des `aktion`-Feldes in den richtigen Google Sheets Node.

**14 · Testen mit allen vier Fällen**
Testet nacheinander: neues Angebot → zweites Angebot vom gleichen Fahrer → überarbeitetes Angebot vom gleichen Anbieter → Angebot wenn alle Slots bereits belegt sind.

---

## Stufe 5 – Vollautomatisch per E-Mail 🟡 Bonus

**Ziel:** Der Workflow läuft ohne manuellen Eingriff. Eingehende E-Mails mit PDF-Anhang werden automatisch verarbeitet.

### Aufgaben

**15 · Mail Trigger konfigurieren**
Ersetzt den manuellen Trigger durch einen Gmail Trigger oder IMAP Trigger. Filtert auf E-Mails mit PDF-Anhang.

**16 · Attachment herunterladen**
Nutzt den Gmail Node um den PDF-Anhang als Binärdatei zu laden und in den Workflow weiterzugeben.

**17 · End-to-End testen**
Schickt eine Test-E-Mail mit einem Leasing-PDF und prüft ob der Workflow vollständig durchläuft und die Tabelle korrekt aktualisiert wird.

---

## Tipps

**HTTP Request → Binary** — Stellt sicher dass der Response Format auf `File` gesetzt ist. Nur dann bekommt der folgende AI Node eine Binärdatei die er als PDF interpretieren kann.

**Verschiedene PDFs testen** — Ändert einfach die Nummer in der URL (`-01` bis `-34`) um verschiedene Angebote zu laden. Jede Nummer ist ein anderer Fahrer oder Anbieter.

**AI Prompt für PDF-Extraktion** — Seid präzise welche Felder ihr wollt und in welchem Format. Beispiel: *„Extrahiere folgende Felder als JSON: fahrer, modell, finanzfaktor (nur die Zahl ohne %), ..."*

**Lookup Zeilennummer** — Der Google Sheets Lookup gibt die Zeilennummer zurück. Diese braucht ihr zwingend für den Update Node.

**Datum parsen** — Für Stufe 4 (ältesten Slot finden) wandelt die Datumswerte mit `new Date(...)` in vergleichbare Objekte um bevor ihr sie vergleicht.

**Stufe 5: Robustheit** — Nicht alle E-Mails haben einen PDF-Anhang. Baut einen IF Node ein der prüft ob ein Attachment vorhanden ist bevor ihr weiterarbeitet.

---

*n8n Training · Use Case 01 · Fuhrpark Leasing-Angebote · Stufen 1–3 Pflicht · Stufen 4–5 Bonus*
