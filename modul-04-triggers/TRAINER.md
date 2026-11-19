# Modul 04 – Triggers – Erklärungen

Was der Trainer am Beamer erklärt. Die Aufgaben stehen in der [README.md](README.md).

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
