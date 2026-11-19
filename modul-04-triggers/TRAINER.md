# Modul 04 – Triggers – Erklärungen

Was der Trainer am Beamer erklärt. Die Aufgaben stehen in der [README.md](README.md).

## Ablauf am Beamer (10 min) – Playground links im Workflow

Alle Nodes in **Zone 1 – Node Playground** (Notiz *Playground – Übersicht*). Trigger lassen sich mit **Execute step** einmal testweise auslösen bzw. auf ein Ereignis warten lassen.

| Schritt | Node | Was zeigen |
|---|---|---|
| 1 | **Schedule Trigger (Playground)** | Zeit-basiert: Intervall und Cron. Läuft von selbst erst, wenn der Workflow aktiv ist |
| 2 | **Webhook (Playground)** | Push: Test-URL und Production-URL, Methode. *Execute step* wartet auf einen Aufruf |
| 3 | **On form submission (Playground)** | Push: n8n baut das Formular selbst, Test-URL im Browser öffnen |
| 4 | **Workflow Input Trigger (Playground)** | Push von innen: ein anderer Workflow ruft diesen auf |
| 5 | **Gmail Trigger (Playground)**, **Google Drive Trigger (Playground)**, **Google Sheets Trigger (Playground)** | Polling: n8n fragt regelmäßig nach. Deshalb immer mit Verzögerung |
| 6 | (kein Node) | Chat Trigger gibt es nur einmal pro Workflow, er kommt in Aufgabe 4 |

Faustregel zum Mitnehmen: **Push reagiert sofort, Polling mit Verzögerung.**

Danach: Workflow in `<Name> – Modul 04 – …` umbenennen lassen, dann Zone 2.

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
⚠️ Pro Workflow ist nur ein Manual Trigger erlaubt – deshalb ist er hier im Playground nicht abgebildet. Ihr kennt ihn aus Modul 0 bis 3.

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

Deshalb werden hier alle Trigger so gut wie manuell ausgelöst: Du konfigurierst jeden Trigger und löst ihn einmal bewusst aus. Jeder Trigger trägt sich dabei selbst ins Scoreboard ein. Das Ziel ist das Kennenlernen und Verstehen der verschiedenen Trigger-Typen – nicht der produktive Einsatz.
