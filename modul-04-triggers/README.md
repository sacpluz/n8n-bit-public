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
## "Wer zuerst alle 6 Trigger zum Feuern bringt gewinnt!"

### ⚠️ Vor dem Start – einmalig erledigen:
Öffne jeden **Edit Fields Node** hinter den Triggern und ersetze
**'HIER_NAMEN_EINTRAGEN'** durch deinen Vor- und Nachnamen. **(Ausgenommen Aufgabe 1 hier muss euer Name im Formular unter Name eingetragen werden)**
Ohne gleicher Schreibweise des Namen wird dein Ergebnis nicht dem Scoreboard zugeordnet!

### Deine Aufgabe:
Konfiguriere alle 5 Trigger-Nodes korrekt und bringe jeden zum Feuern.
Jeder Trigger schreibt seinen Eintrag ins Google Sheet (Part 4 Data).
Wenn du alle 5 ausgeführt hast, betätige Aufgabe 6 – der manuelle Trigger
fasst alles zusammen und überträgt dein Ergebnis ins Scoreboard.

### ⚠️ Reihenfolge einhalten – Aufgaben der Reihe nach abarbeiten!
Starte erst mit Aufgabe 2 wenn Aufgabe 1 erfolgreich war.
Aufgabe 6 erst ausführen wenn alle 5 Trigger gefeuert haben.

### Die 6 Trigger:
1. ⏱️ **Schedule** → alle 10 Sekunden (Workflow aktivieren!)
2. 🌐 **Webhook** → POST auf `/briefing` (Production URL per Mail an Facilitator)
3. 📝 **Formular** → 3 Felder (Name, Abteilung, Nachricht)
4. 💬 **Chat** → beliebige Nachricht
5. 📧 **Gmail** → bei neuer E-Mail an die Trainings-Adresse
6. 🖱️ **Manuell** → Wenn alle 5 erledigt sind: Workflow ausführen und Ergebnis einreichen

### 🏆 Gamification:
Der **Timestamp** der manuellen Einreichung (Aufgabe 6) entscheidet.
**Korrektheit geht vor Geschwindigkeit** — alle 5 Trigger müissen gefeuert haben.

---

### AUFGABE 1 – Formular
📝 Erstelle ein Formular mit 3 Feldern.

Pflicht:
- Name (Text) => **Deinen Vor- und Nachname eintragen**  ⚠️  ⚠️  ⚠️ 
- Abteilung (Text)
- Nachricht (Textarea)

Tipp: n8n generiert eine Live-URL – öffne sie im Browser und fülle das Formular aus!

---

### 🎯 AUFGABE 2 – Webhook
Konfiguriere einen Webhook der auf POST-Anfragen reagiert.

- **Path**: challenge-webhook
- **Method**: POST

Der Facilitator-Workflow schickt alle 10 sekunden etwas gegen diesen Webook. Der Faciliator-Workflow wird vom Workshop-Moderator gesteuert. Kein To Do für dich.

---

### AUFGABE 5 – Gmail Trigger
📧 Konfiguriere den Gmail Trigger.

**Poll**: Jede Minute
**Filter**: Bereits definiert prüfe was definiert wurde
**Sender**: Deine BridgingIT Mail-Adresse eintragen. Beispiel vorname.nachname@bridging-it.de

Sende nach dem Setup der Node eine Nachricht an n8n.training.bridgingit@gmail.com sodass die Mail vom Trigger erkannt wird.


