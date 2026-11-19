# n8n Workshop – bridgingIT

Handouts und Workflow-Vorlagen für den hands-on n8n-Workshop.

> 💡 Lasst das Handout des aktuellen Moduls in einem **zweiten Browser-Tab** offen. Code-Blöcke haben oben rechts einen Kopieren-Knopf.

## So arbeitet ihr

1. **Vor dem Workshop:** [Credentials anlegen](modul-00-credentials/README.md) (10 min).
2. Pro Modul gibt es ein **Handout** (`README.md`) und eine **Workflow-Vorlage** (`.json`). Die Vorlage liegt meist schon in eurem Ordner in n8n. Sonst: in n8n neuer Workflow → Menü `⋯` → **Import from File**.
3. Der Node **Ergebnis prüfen** sagt euch im Feld `hinweise`, was noch nicht stimmt.
4. Euer Name fürs Scoreboard steht in **AUFGABE 0**, dem ersten Node jedes Workflows (Modul 4: im Workflow-Namen). Wie gewertet wird: [Scoreboard](scoreboard.md).

## Module

| # | Modul | Handout |
|---|---|---|
| vorab | Credentials anlegen | [modul-00-credentials](modul-00-credentials/README.md) |
| 0 | Grundlagen: UI, JSON & Data Mapping | [modul-00-grundlagen](modul-00-grundlagen/README.md) |
| 1 | Data 1: Lead Qualification | [modul-01-data-1](modul-01-data-1/README.md) |
| 2 | Data 2: Datenqualität | [modul-02-data-2](modul-02-data-2/README.md) |
| 3 | Workflows steuern | [modul-03-workflows-steuern](modul-03-workflows-steuern/README.md) |
| 3.1 | Looping (Bonus) | [modul-03-1-looping-bonus](modul-03-1-looping-bonus/README.md) |
| 4 | Triggers | [modul-04-triggers](modul-04-triggers/README.md) |
| 5 | Error Handling | [modul-05-error-handling](modul-05-error-handling/README.md) |
| 6 | Files | [modul-06-files](modul-06-files/README.md) |
| 7 | AI + Model | [modul-07-ai-model](modul-07-ai-model/README.md) |
| 8 | AI + Structured Data | [modul-08-ai-structured-data](modul-08-ai-structured-data/README.md) |
| 9 | Tools und DBs | [modul-09-tools-dbs](modul-09-tools-dbs/README.md) |

## Weiterlesen

- [Enterprise-Ausblick](enterprise-ausblick.md): Was es braucht, damit Workflows im Unternehmen produktiv laufen.
- [Scoreboard](scoreboard.md): Regeln des Wettbewerbs.

## Use Cases

Zum Abschluss baut ihr im Team einen der fünf Use Cases, mit Dummy-Daten zum Ausprobieren.

| # | Use Case | Beschreibung |
|---|---|---|
| 01 | [Fuhrpark](use-cases/01-fuhrpark/README.md) | Leasing-Angebote aus PDFs auslesen und in einer Übersicht pflegen |
| 02 | [Bauhaus](use-cases/02-bauhaus/README.md) | App-Bewertungen auswerten, beantworten und Feature-Wünsche als Jira-Ticket anlegen |
| 03 | [Abrechnung](use-cases/03-abrechnung/README.md) | Stunden prüfen, mit dem Forecast vergleichen und Rechnungen als PDF erzeugen |
| 04 | [Bewerber](use-cases/04-bewerber/README.md) | Bewerbungsunterlagen auslesen und mit der Stellenbeschreibung abgleichen |
| 05 | [Claude + MCP + n8n](use-cases/05-claude+mcp+n8n/README.md) | n8n-Workflows per Chat beschreiben und von Claude Code bauen lassen |
