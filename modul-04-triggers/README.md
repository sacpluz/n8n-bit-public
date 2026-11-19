# Modul 04 – Triggers

Dieses Handout ist der Text der Sticky Notes aus dem Workflow, damit ihr ihn im zweiten Browser-Tab lesen und Code kopieren könnt.

**Workflow-Datei:** [`modul-04-triggers.json`](modul-04-triggers.json)

---

# 🎯 ZONE 1 – NODE PLAYGROUND
### Triggers & Scheduling

## Die vier Wege wie ein Workflow starten kann:

⏱️ **ZEIT-BASIERT**
Der Workflow startet automatisch zu einem definierten Zeitpunkt oder Intervall – ohne dass ein Mensch etwas tut.
→ Schedule Trigger (Cron, stündlich, täglich, alle X Sekunden)

🖱️ **MANUELL**
Ein Mensch startet den Workflow bewusst per Klick direkt in n8n – kein externes Ereignis, kein Zeitplan.
→ Manual Trigger (der "Execute workflow"-Button)
⚠️ Pro Workflow ist nur ein Manual Trigger erlaubt – deshalb ist er hier im Playground nicht abgebildet. Er kommt in Zone 2 als Aufgabe 6 vor.

🌐 **EVENT-BASIERT (Push)**
Ein externes, internes System oder ein Mensch schickt aktiv etwas an n8n – n8n wartet passiv bis etwas ankommt.
→ Webhook (HTTP-Anfrage von außen)
→ On form submission (jemand füllt ein Formular aus)
→ When chat message received (jemand schreibt im Chat)
→ Workflow Input Trigger (ein anderer n8n Workflow sendet an diesen Trigger)
→ Produktspezifische Trigger: z.B. Calendly, Gumroad, Stripe – jeder Dienst kann bei bestimmten Ereignissen einen Webhook feuern

🔄 **POLLING (Pull)**
n8n fragt selbst regelmäßig bei einem Dienst nach ob etwas Neues da ist – kein Push von außen.
→ Gmail Trigger (prüft jede Minute das Postfach)
→ Google Sheets Trigger (prüft ob neue Zeilen erschienen sind)
→ Viele weitere App-Trigger funktionieren nach diesem Prinzip

💡 Faustregel: Push-Trigger reagieren sofort. Polling-Trigger haben immer eine Verzögerung.

---
⚠️ **Hinweis für diesen Workshop:**
In einem echten Produktiv-Workflow hat jeder Workflow genau einen Trigger – er reagiert auf ein bestimmtes Ereignis und läuft dann durch.

In diesem Modul wollen wir jedoch alle Trigger-Typen in einem einzigen Workflow demonstrieren – das ist technisch gesehen nicht der vorgesehene Einsatz. n8n ist nicht darauf ausgelegt, mehrere Trigger gleichzeitig innerhalb eines Workflows zu kombinieren und deren Ergebnisse zu sammeln.

Deshalb werden hier alle Trigger so gut wie manuell ausgelöst: Du konfigurierst jeden Trigger, löst ihn einmal bewusst aus und bestätigst am Ende mit Aufgabe 6 dass alle durchgelaufen sind. Das Ziel ist das Kennenlernen und Verstehen der verschiedenen Trigger-Typen – nicht der produktive Einsatz.

---

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


