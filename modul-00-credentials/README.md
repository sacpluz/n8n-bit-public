# Modul 0 – Credentials in einem Rutsch anlegen

**Dauer:** 10 Min · Einmalig vor dem Workshop · Jede Person in der eigenen Instanz

---

## 🎯 Ziel

Alle Zugangsdaten, die die späteren Module brauchen, werden **einmal** in einen Workflow eingetragen. Beim Ausführen legt der Workflow sie als Credentials in deiner n8n-Instanz an. Du musst sie nicht einzeln über die Oberfläche erstellen.

Genutzt wird der **n8n-Node** (Resource `Credential`, Operation `Create`). Er spricht die Public API deiner Instanz an.

---

## 📦 Enthaltene Credentials

| Name im Workflow | Typ | Gebraucht in | Felder |
|---|---|---|---|
| Workshop – Google Gemini | `googlePalmApi` | AI-Module (Part 07) | `apiKey` |
| Workshop – OpenAI | `openAiApi` | AI-Module | `apiKey` |
| Workshop – Qdrant | `qdrantApi` | RAG / Vektor-Use-Case | `qdrantUrl`, `apiKey` |
| Workshop – Airtable | `airtableTokenApi` | Apps & Integrations | `accessToken` |
| Workshop – GitHub | `githubApi` | Apps & Integrations | `user`, `accessToken` |
| Workshop – Ollama | `ollamaApi` | lokale Modelle | `baseUrl` |

Nicht ausgefüllte Einträge (Wert beginnt mit `HIER_EINTRAGEN`) werden **übersprungen**. Du trägst also nur ein, was du wirklich brauchst.

> **Nicht enthalten:** Google Sheets und Google Drive (OAuth2). Die Public API legt nur eine leere Hülle an, die Autorisierung bleibt ein Klick in der Oberfläche (siehe Schritt 6).

---

## 🚀 Anleitung

### 1 · Workflow importieren
Neuer Workflow → Menü `⋯` → **Import from File** → [`modul-00-credentials.json`](modul-00-credentials.json).

### 2 · API-Key in n8n erzeugen
**Settings → n8n API → Create an API key** (Label z. B. `workshop`, Ablauf nach Bedarf). Key kopieren.

### 3 · n8n-API-Credential verbinden
Node **Credentials anlegen** öffnen → bei *Credential to connect with* **Create new credential**:

- **API Key:** der Key aus Schritt 2
- **Base URL:** `https://<deine-instanz>/api/v1` (lokal z. B. `http://localhost:5678/api/v1`)

Speichern. Der Verbindungstest muss grün werden.

> **n8n läuft im Docker/Devcontainer?** Dann statt `localhost` den Containernamen oder `http://host.docker.internal:5678/api/v1` verwenden.

### 4 · Zugangsdaten eintragen
Node **Zugangsdaten eintragen** öffnen und jedes `HIER_EINTRAGEN` durch den echten Wert ersetzen. Nicht benötigte Blöcke einfach unverändert lassen.

### 5 · Ausführen
**Execute workflow** klicken. Der Node **Ergebnis** zeigt pro Credential `angelegt` mit ID oder `FEHLER` mit Meldung (die Secrets werden nicht ausgegeben).

### 6 · OAuth-Credentials per Hand
Für Google Sheets / Drive: **Credentials → Create credential → Google Sheets OAuth2 API** → *Sign in with Google*. Das brauchen nur Modul 5 und 6. Fürs Scoreboard braucht ihr keine eigene, das läuft über die Credential des Facilitators.

### 7 · Aufräumen ⚠️
- Werte im Code-Node wieder auf `HIER_EINTRAGEN` zurücksetzen.
- Den Workflow **nicht** mit eingetragenen Keys speichern, exportieren oder teilen. Die Secrets stehen sonst im Klartext im Workflow und im Execution-Log.
- Den API-Key aus Schritt 2 danach löschen (Settings → n8n API).

---

## 🧪 Kontrolle

- **Credentials**-Übersicht zeigt die sechs Einträge `Workshop – …` (soweit befüllt).
- Beim Öffnen einer Credential sind die Secret-Felder maskiert.
- In einem späteren Modul lässt sich die Credential im Dropdown des Nodes auswählen.

---

## 🛠 Fehlersuche

| Meldung | Ursache / Lösung |
|---|---|
| `401 Unauthorized` | API-Key falsch oder abgelaufen (Schritt 2/3) |
| `ECONNREFUSED` / Timeout | Base URL falsch, bei Docker siehe Hinweis oben |
| `request.body.data … is required` | Pflichtfeld des Typs fehlt. Schema abrufen: n8n-Node → Resource `Credential` → Operation **Get Schema** → Typ z. B. `openAiApi` |
| `Credential with name already exists` o. ä. | Credential existiert schon. In der Oberfläche löschen oder den Namen im Code-Node ändern |
| Keine Ausgabe | Alles noch `HIER_EINTRAGEN`, daher wurde nichts angelegt |

---

## ℹ️ Hintergrund

- Die Public API kann Credentials nur **anlegen** und **löschen**, nicht ändern. Zum Aktualisieren löschen und neu anlegen. Die ID ändert sich dabei, Workflows müssen dann neu zugeordnet werden.
- Alternative ohne Workflow, für Self-Hosted: `n8n import:credentials --input=creds.json`.
