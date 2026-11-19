# Beispiele zum Weiterbauen

Fertige Workflows, die über die Module hinausgehen: MCP, Human in the Loop, mehrere Agenten, parallele Abläufe. Keine Aufgaben, keine Wertung. Importiert sie, schaut sie euch an und baut sie für eure eigenen Fälle um.

## So nutzt ihr sie

1. JSON-Datei herunterladen (oder den Inhalt kopieren) und in n8n über **Import from File** bzw. per Einfügen in einen leeren Workflow importieren.
2. Credentials sind **nicht** enthalten. Öffnet jeden Node mit rotem Warnzeichen und wählt eure eigenen Credentials aus.
3. Felder mit Platzhaltern füllen: leere Auswahlfelder (Sheet, Kalender, Airtable-Base, Repository, Sub-Workflow), `vorname.nachname@bridging-it.de` und `https://DEINE-INSTANZ.app.n8n.cloud/mcp/DEIN-MCP-PFAD`.
4. Die gelben Notizen im Workflow erklären, wie er funktioniert.

## Übersicht

| Datei | Was er zeigt | Braucht |
|---|---|---|
| [mcp-01-agent.json](mcp-01-agent.json) | AI Agent holt sich seine Tools über einen MCP Client statt direkt angebunden. Notiz: Direkt vs. MCP, wann was? | Gemini, URL eines MCP Servers (z. B. aus `mcp-02-server.json`) |
| [mcp-02-server.json](mcp-02-server.json) | Ein n8n-Workflow als MCP Server, der ein Gmail-Tool anbietet | Gmail |
| [mcp-03-details.json](mcp-03-details.json) | Ein zentraler MCP Server oder einer pro Domäne (Gmail, Drive, Jira)? Beide Varianten nebeneinander | Gemini, Gmail, Google Drive, Jira |
| [human-in-the-loop-email.json](human-in-the-loop-email.json) | Eingehende E-Mail zusammenfassen, Antwort entwerfen, vor dem Senden freigeben lassen. Freigabe-Varianten für Gmail, Slack, Teams, Telegram, Discord, WhatsApp u. a. | OpenAI, IMAP/SMTP oder Gmail, ein Kanal für die Freigabe |
| [human-in-the-loop-termin.json](human-in-the-loop-termin.json) | Nach vergangenen Terminen prüfen, ob schon nachgefasst wurde. Agent schlägt Folgetermin vor, ihr gebt per Mail frei, dann wird gebucht | OpenAI, Gmail, Google Calendar |
| [multi-agent.json](multi-agent.json) | Ein Haupt-Agent verteilt Aufgaben an Unter-Agenten für E-Mail, Kalender, Kontakte und Inhalte (Steuerung per Telegram) | Telegram, OpenAI/OpenRouter, Gmail, Google Calendar, Airtable, Tavily |
| [parallelisierung.json](parallelisierung.json) | Recherche zu mehreren Tools läuft parallel, die Auswertung danach nacheinander | OpenRouter, Perplexity, Google Sheets |
| [n8n-backup-github.json](n8n-backup-github.json) | Alle Workflows eines n8n-Projekts als JSON in ein GitHub-Repo sichern | n8n-API-Key, GitHub |

## Hinweise

- **Multi Agent:** Die Unter-Agenten sind als eigene Workflows gedacht. Laut Setup-Notiz im Workflow kopiert ihr jeden Agenten in einen eigenen Workflow und wählt ihn dann im passenden Tool-Node des Haupt-Agenten aus.
- **MCP Server** sind nur erreichbar, solange der Workflow aktiv ist (oder im Test-Modus läuft). Die URL steht im Node `MCP Server Trigger`.
- **Herkunft:** `multi-agent.json` und `parallelisierung.json` stammen von [Nate Herk](https://www.youtube.com/@nateherk), `human-in-the-loop-termin.json` beruht auf einer Vorlage aus der n8n-Template-Galerie. Die Notizen der Autoren sind im Workflow geblieben.
