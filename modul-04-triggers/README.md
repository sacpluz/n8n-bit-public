# Modul 04 – Triggers

Dieses Handout ist der Text der Sticky Notes aus dem Workflow, damit ihr ihn im zweiten Browser-Tab lesen und Code kopieren könnt.

**Workflow-Datei:** [`modul-04-triggers.json`](modul-04-triggers.json)

**Erklärungen zum Modul** (was der Trainer am Beamer zeigt, zum Nachlesen): [TRAINER.md](TRAINER.md)

---

# 🏋️ ZONE 2 – CHALLENGE
## "Wer zuerst alle 5 Trigger zum Feuern bringt, gewinnt!"

### ⚠️ Vor dem Start – einmalig:
Workflow umbenennen: oben links auf den Namen klicken und **VORLAGE** durch euren Vor- und Nachnamen ersetzen, z. B. `Anna Müller – Modul 04 – Triggers`.
Daraus liest jeder Trigger euren Namen fürs Scoreboard. In den Edit-Fields-Nodes nichts eintragen.

### Eure Aufgabe:
Konfiguriert alle 5 Trigger-Nodes und bringt jeden einmal zum Feuern.
Jeder Trigger trägt sich selbst ins Scoreboard ein.

### Die 5 Trigger:
1. ⏱️ **Schedule** → alle 10 Sekunden (Workflow aktivieren!)
2. 🌐 **Webhook** → POST auf `/briefing` (Production URL per Mail an Facilitator)
3. 📝 **Formular** → 3 Felder (Name, Abteilung, Nachricht)
4. 💬 **Chat** → beliebige Nachricht
5. 📧 **Gmail** → bei neuer E-Mail an die Trainings-Adresse

### 🏆 Wertung:
Es gewinnt, wer zuerst alle 5 Trigger ausgelöst hat. Jeder Trigger zählt pro Person einmal.

---

### AUFGABE 1 – Formular
📝 Erstelle ein Formular mit 3 Feldern.

Pflichtfelder:

| Field Label | Element Type | Hinweis |
|---|---|---|
| Name | Text | |
| Abteilung | Text | |
| Nachricht | Textarea | |

Tipp: n8n generiert eine Live-URL – öffne sie im Browser und fülle das Formular aus!

---

### 🎯 AUFGABE 2 – Webhook
Konfiguriere einen Webhook der auf POST-Anfragen reagiert.

| Einstellung | Wert |
|---|---|
| HTTP Method | `POST` |
| Path | `challenge-webhook` |

Der Facilitator-Workflow schickt alle 10 sekunden etwas gegen diesen Webook. Der Faciliator-Workflow wird vom Workshop-Moderator gesteuert. Kein To Do für dich.

---

### AUFGABE 5 – Gmail Trigger
📧 Konfiguriere den Gmail Trigger.

| Einstellung | Wert |
|---|---|
| Poll Times | Every Minute |
| Filter | schon gesetzt, prüfe was drinsteht |
| Sender | deine bridgingIT-Adresse, z. B. `vorname.nachname@bridging-it.de` |

Sende nach dem Setup der Node eine Nachricht an n8n.training.bridgingit@gmail.com sodass die Mail vom Trigger erkannt wird.

