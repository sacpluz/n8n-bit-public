# Use Case 05 – Workflow-Builder-Agent: n8n-Workflows per Chat beschreiben und automatisch bauen lassen

## Szenario

Du öffnest VS Code, beschreibst in einem Chat-Fenster was du automatisieren möchtest — in normaler Sprache. Claude Code übersetzt deine Beschreibung in einen vollständigen n8n-Workflow, verbindet sich direkt mit deiner n8n-Instanz, und Sekunden später taucht der fertige Workflow in deinem Canvas auf. Kein manuelles Nodes-Setzen, kein JSON-Tippen — nur beschreiben und bauen lassen.

Der Trick dahinter: Claude Code kennt durch den **n8n MCP Server** alle Node-Typen, Parameter und Verbindungsregeln. Durch die **n8n Skills** weiß es zusätzlich wie es denken muss wenn es einen Workflow strukturiert. Ohne diese beiden Add-ons liegt die Erfolgsquote bei ~40%. Mit ihnen bei ~90–95%.

---

## Gesamtarchitektur

```
Du (natürliche Sprache)
        ↓
Claude Code (VS Code Extension)
  ├── claude.md       → Systemkontext und Verhaltensregeln
  ├── n8n Skills      → Denkweise: wie baut man gute Workflows?
  └── n8n MCP Server  → Wissen: alle Nodes, Parameter, Typen
        ↓ (direkte API-Verbindung)
n8n Instanz
  → Workflow wird live erstellt und ist sofort nutzbar
```

**Was Claude Code anders macht als ein normaler LLM-Prompt:**
Claude Code ist kein einfacher Chatbot. Es kann Dateien lesen und schreiben, Terminals ausführen, GitHub-Repos klonen, APIs aufrufen — und dabei selbst entscheiden wann es welches Tool braucht. Der n8n MCP Server ist sein direkter Kanal zur n8n-Instanz.

---

## Voraussetzungen

| Was | Warum |
|---|---|
| **Claude Pro / Teams / Enterprise** | Claude Code ist nicht im Free Plan enthalten (ab $20/Monat) |
| **VS Code** | Platform auf der Claude Code läuft |
| **n8n Instanz** | Lokal oder Cloud — braucht eine erreichbare URL |
| **n8n API Key** | Damit Claude Code in die Instanz schreiben darf |

---

## Stufe 1 – VS Code und Claude Code einrichten 🟢 Pflicht

**Ziel:** Claude Code läuft in VS Code und ist mit eurem Claude-Account verbunden.

### Aufgaben

**01 · VS Code installieren**

Download unter: `code.visualstudio.com`

Nach der Installation VS Code öffnen — ihr seht eine leere Startoberfläche.

**02 · Claude Code Extension installieren**

In VS Code links in der Sidebar auf das Extensions-Symbol klicken (vier Quadrate). Nach `Claude Code` suchen. Erste Ergebnis auswählen → Install drücken.

Nach erfolgreicher Installation: Uninstall- und Disable-Button sichtbar → alles korrekt.

**03 · Arbeitsordner anlegen**

Auf dem Desktop einen neuen Ordner anlegen, z.B. `n8n-agent-builder`.

In VS Code: `Open Folder` → Desktop → `n8n-agent-builder` auswählen → `Yes, I trust the authors`.

**04 · Claude Code Interface öffnen**

Oben rechts in VS Code auf das Chat-Symbol klicken. Das Claude Code Interface öffnet sich — es sieht aus wie ein normaler Chat.

**05 · Bypass-Permission aktivieren**

In VS Code: `Settings` (Zahnrad) → nach `Claude Code` suchen → **Enable Bypass Permission Mode** aktivieren.

> Was bedeutet das? Claude Code hat vier Modi:
> - **Bypass Permission** (rot) — führt alles ohne Nachfragen aus, ideal für unbeaufsichtigtes Arbeiten
> - **Ask before edits** — fragt vor jeder Änderung nach
> - **Edit automatically** — führt Änderungen automatisch aus, fragt aber bei großen Schritten
> - **Plan mode** (blau) — zeigt zuerst den kompletten Plan, ideal zum Lernen und Verstehen

Für den Einstieg: **Plan mode** verwenden. Man sieht wie Claude Code denkt. Für erfahrene Nutzer: Bypass Permission aktivieren und loslaufen lassen.

### Erwartetes Ergebnis

Claude Code ist installiert, der Arbeitsordner ist geöffnet, das Chat-Interface ist aktiv. Ihr könnt bereits Nachrichten eingeben — aber Claude Code weiß noch nichts über n8n.

---

## Stufe 2 – n8n API verbinden 🟢 Pflicht

**Ziel:** Claude Code bekommt Zugriff auf eure n8n-Instanz.

### Aufgaben

**06 · n8n API Key erstellen**

In n8n: `Settings` → `n8n API` → `Create an API key`.

Name: z.B. `Claude Code`. Speichern. Den Key kopieren — er wird nur einmal angezeigt.

**07 · n8n Instanz-URL notieren**

Die URL eurer n8n-Instanz aus der Browserzeile kopieren — alles bis zum ersten `/` nach dem Domain-Namen.

Beispiele:
```
Lokal:  http://localhost:5678
Cloud:  https://meine-instanz.n8n.cloud
```

**08 · Beide Werte bereithalten**

API Key und URL werden in Stufe 3 von Claude Code automatisch in die Konfiguration eingebaut. Ihr müsst nur bereit sein sie einzufügen wenn Claude Code danach fragt.

### Erwartetes Ergebnis

Ihr habt API Key und URL zur Hand. n8n läuft und ist erreichbar.

---

## Stufe 3 – n8n MCP Server und n8n Skills installieren 🟢 Pflicht

**Ziel:** Claude Code lernt n8n kennen. Das ist der entscheidende Schritt — ohne diesen Setup liegt die Trefferquote bei ~40%.

**Was installiert wird:**
- **n8n MCP Server** — alle Node-Typen, Parameter, Verbindungsregeln. Das *Wissen* über n8n.
- **n8n Skills** — die Denkweise wie man einen guten Workflow strukturiert. Das *Können*.

### Aufgaben

**09 · claude.md Datei anlegen**

Im VS Code Explorer (links) eine neue Datei anlegen: `claude.md`

Diese Datei ist der Systemkontext für Claude Code — ähnlich einem System-Prompt. Sie bleibt in eurem Arbeitsordner und wird bei jeder Sitzung automatisch geladen.

**10 · Claude Code den Setup durchführen lassen**

Im Claude Code Chat folgendes eingeben:

```
Ich richte dieses Projekt ein um n8n-Workflows und AI-Agenten 
per Prompt zu bauen. Du bekommst Zugriff auf zwei Dinge:

1. n8n Skills (GitHub): https://github.com/n8n-io/n8n-mcp
2. n8n MCP Server (GitHub): [URL aus dem Video / vom Trainer]

Installiere beides, konfiguriere den MCP Server mit meiner 
n8n-Instanz-URL und meinem API Key, und befülle die claude.md 
mit einem passenden Systemkontext für dieses Projekt.

n8n URL: [eure URL]
n8n API Key: [euer Key]
```

> **Plan mode nutzen:** Ihr seht Schritt für Schritt was Claude Code macht — GitHub-Repos klonen, Dateien anlegen, MCP konfigurieren. Claude Code stellt möglicherweise Rückfragen: Erfahrungslevel mit n8n (`advanced`), Validierungsverhalten (`always validate strictly`).

**11 · Installation bestätigen**

Claude Code zeigt am Ende eine Zusammenfassung was installiert wurde:
- n8n Skills ✓
- n8n MCP Server ✓
- mcp.json konfiguriert ✓
- claude.md befüllt ✓

**12 · Verbindung testen**

Im Claude Code Chat:
```
Zeige mir alle n8n-Workflows in meiner Instanz.
```

Wenn eine Liste (oder eine leere Liste) zurückkommt — Verbindung funktioniert. Wenn eine Fehlermeldung kommt — API Key oder URL prüfen.

### Erwartetes Ergebnis

Claude Code hat Zugriff auf eure n8n-Instanz und kennt alle n8n-Nodes. Der Arbeitsordner enthält `claude.md`, `mcp.json` und die installierten Skills. Dieser Setup muss nur einmal gemacht werden.

---

## Stufe 4 – Ersten Workflow per Prompt bauen 🟢 Pflicht

**Ziel:** Claude Code baut einen echten Workflow in eurer n8n-Instanz — von der Beschreibung bis zum fertigen Canvas.

### Einstiegs-Prompts (von einfach nach komplex)

Gebt diese Beschreibungen nacheinander ein. Startet mit dem einfachsten:

---

**Prompt 1 — Minimal (3 Nodes)**
```
Erstelle einen Workflow der manuell gestartet werden kann, 
das aktuelle Datum und die Uhrzeit berechnet und das Ergebnis 
als Feld 'timestamp' ausgibt.
```

Erwartetes Ergebnis: `Manual Trigger → Code Node → Set Node`

---

**Prompt 2 — Etwas komplexer (5 Nodes)**
```
Erstelle einen Workflow der täglich um 8 Uhr morgens startet, 
eine HTTP-Anfrage an https://api.github.com/zen schickt 
(ein zufälliges Zitat), und das Ergebnis in eine 
Google-Sheets-Tabelle einträgt.
```

Erwartetes Ergebnis: `Schedule Trigger → HTTP Request → Set Node → Google Sheets`

---

**Prompt 3 — Mit Entscheidungslogik (7+ Nodes)**
```
Erstelle einen Webhook-Workflow der eingehende Daten empfängt. 
Das Feld 'score' wird geprüft: Leads mit score >= 60 bekommen 
status = 'qualified', alle anderen status = 'rejected'. 
Beide Pfade werden am Ende zusammengeführt und als 
JSON-Response zurückgegeben.
```

Erwartetes Ergebnis: `Webhook → IF → Set (qualified) → Merge → Respond to Webhook`
                                 `→ Set (rejected) → ↗`

### Aufgaben

**13 · Prompt eingeben und Plan beobachten**

Plan mode nutzen. Claude Code zeigt seinen Denkprozess: Nodes auswählen, Verbindungen planen, Parameter setzen.

**14 · Workflow in n8n öffnen**

Claude Code gibt nach dem Erstellen einen Link aus. Link öffnen → Workflow erscheint direkt im Canvas.

**15 · Workflow manuell prüfen**

Was stimmt? Was fehlt? Typische Lücken die Claude Code offen lässt:
- Credentials nicht verbunden (Google Sheets Auth, Gmail Auth)
- Dropdown-Felder leer (Sheet ID, Kalender-ID)
- Webhook-URL muss aktiviert werden

Diese 5–10% müssen manuell ergänzt werden — dafür braucht ihr eure n8n-Kenntnisse.

### Erwartetes Ergebnis

Ein Workflow taucht im n8n Canvas auf. Struktur und Node-Typen stimmen. Einzelne Felder die manuelle Auswahl erfordern sind leer — das ist normal und erwartet.

---

## Stufe 5 – Komplexen Multi-Step AI-Agenten bauen 🟡 Bonus

**Ziel:** Claude Code baut einen vollständigen AI-Agenten mit mehreren Sub-Workflows — genau wie im Video.

### Beispiel-Prompt aus dem Video

```
Baue einen AI-Agenten mit Telegram-Trigger als Eingabe.
Der Haupt-Agent soll drei Sub-Agenten koordinieren:

1. E-Mail-Agent (Gmail): Schreibt und sendet E-Mails
2. Kalender-Agent (Google Calendar): Erstellt Termine
3. Recherche-Agent (Perplexity/HTTP): Recherchiert Themen

LLM: OpenAI GPT-4
Der Haupt-Agent entscheidet selbst welcher Sub-Agent 
für eine Anfrage zuständig ist.
Antwort geht zurück an Telegram.
```

> **Erwartung setzen:** Ein Workflow dieser Komplexität (40–60+ Nodes, 4 Sub-Workflows) dauert 5–10 Minuten. Claude Code baut ~90% korrekt — den Rest müsst ihr kennen.

### Was Claude Code dabei selbst macht

- Mehrere separate Workflows anlegen (Haupt + 3 Sub)
- Execute Workflow Trigger in jedem Sub-Workflow konfigurieren
- Werkzeug-Verlinkung zwischen Haupt-Agent und Sub-Agenten aufbauen
- AI Agent Node mit Tool-Nodes verbinden
- Rückkanal zum Telegram-Node einrichten

### Was ihr manuell ergänzen müsst

- Credentials für Gmail, Google Calendar, Telegram, OpenAI hinterlegen
- Sub-Workflows aktivieren (Status: Active)
- `Accept All Data` in den Execute Workflow Triggern aktivieren
- Fehlende Prompts in AI Agent Nodes ergänzen

### Erwartetes Ergebnis

Vier verlinkte Workflows im n8n Canvas. Grundstruktur vollständig. Nach 10–15 Minuten manuellem Nacharbeiten ist der Agent testbereit.

---

## Stufe 6 – Workflow per Chat iterativ verbessern 🟡 Bonus

**Ziel:** Anstatt von Null anzufangen, beschreibt ihr Änderungen — Claude Code aktualisiert den bestehenden Workflow.

### Beispiel-Prompts für Iterationen

```
Füge dem Workflow 'Lead Qualification' am Ende einen 
Gmail-Node hinzu der eine Zusammenfassung an 
info@beispiel.de schickt.
```

```
Der IF-Node im Workflow 'Score Check' soll den Schwellenwert 
von 60 auf 75 ändern.
```

```
Ersetze den Manual Trigger im Workflow 'Daily Report' 
durch einen Schedule Trigger der täglich um 7 Uhr startet.
```

### Aufgaben

**16 · Workflow-Namen im Prompt nennen**

Claude Code findet den Workflow über den Namen in eurer Instanz. Name aus n8n genau übernehmen.

**17 · Änderung präzise beschreiben**

Je konkreter die Beschreibung, desto treffsicherer die Änderung. Vage Formulierungen führen zu Rückfragen oder falschen Nodes.

**18 · Diff prüfen**

Nach der Änderung: Canvas neu laden und prüfen ob nur das geändert wurde was ihr beschrieben habt. Claude Code ändert manchmal mehr als nötig.

---

## Was Claude Code kann — und was nicht

| Claude Code kann | Claude Code kann nicht |
|---|---|
| Nodes erstellen und verbinden | Credentials verbinden (OAuth, API Keys) |
| Logik strukturieren (IF, Merge, Loop) | Dropdown-Felder befüllen (Sheet ID, Kalender) |
| AI-Agenten mit Tools aufbauen | Webhooks aktivieren |
| Sub-Workflows anlegen und verlinken | Workflow testen und debuggen |
| Bestehende Workflows updaten | Fehlende Felder erkennen die UI-Auswahl brauchen |
| Komplexe Beschreibungen interpretieren | 100% produktionsfertige Workflows liefern |

**Faustregel aus dem Video:** Claude Code macht ~90% der Arbeit. Die restlichen 5–10% erfordern n8n-Grundkenntnisse. Wer n8n nicht kennt, weiß nicht was fehlt.

---

## Tipps

**Plan mode für den Einstieg** — Immer. Ihr seht wie Claude Code denkt, welche Nodes es wählt und warum. Das ist lehrreicher als das fertige Ergebnis.

**Einfach anfangen** — Startet mit 3-Node-Workflows. Komplex wird es erst wenn die Grundverbindung zur n8n-Instanz sicher funktioniert.

**Rückfragen ernst nehmen** — Claude Code fragt im Plan mode nach wenn es etwas nicht versteht. Klare Antworten führen zu besseren Workflows. `Advanced` als Erfahrungslevel angeben wenn ihr n8n schon kennt.

**Credentials sind immer manuell** — Das ist kein Bug. OAuth-Flows können nicht automatisiert werden. Immer einplanen dass Google Sheets, Gmail, OpenAI etc. manuell verbunden werden müssen.

**API Key sichern** — Den n8n API Key nicht in Dateien committen oder teilen. Er gibt vollen Zugriff auf die n8n-Instanz.

**Bypass Permission mit Bedacht** — Praktisch für unbeaufsichtigtes Arbeiten, aber Claude Code kann dann ohne Rückfrage Dateien anlegen, löschen und ändern. Für Trainingsumgebungen: Plan mode bevorzugen.

---

*n8n Training · Use Case 05 · Claude Code + n8n MCP · Stufen 1–4 Pflicht · Stufen 5–6 Bonus*
*Voraussetzung: Claude Pro ($20/Monat) · VS Code · laufende n8n-Instanz*