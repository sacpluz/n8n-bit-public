# Modul 08 – AI + Structured Data

Dieses Handout ist der Text der Sticky Notes aus dem Workflow, damit ihr ihn im zweiten Browser-Tab lesen und Code kopieren könnt.

**Workflow-Datei:** [`modul-08-ai-structured-data.json`](modul-08-ai-structured-data.json)

---

## 🗺️ AI + Strukturierte Daten – Node-Übersicht

| 4 Nodes | Kategorie: AI Agents |

---

**Was passiert hier?**
Ein RSS-Feed wird abgerufen, bereinigt, von einem LLM analysiert –
und das Ergebnis landet als sauberes JSON in Google Sheets.

---

- **🌐 HTTP Request** – Ruft den RSS-Feed als XML-Rohtext ab. Ohne Custom `User-Agent`-Header blockiert der Server die Anfrage.
- **⚙️ Code** – Bereinigt den XML-Rohtext: entfernt Metadaten, normalisiert HTML-Entities, gibt lesbaren Plaintext zurück.
- **🔗 Basic LLM Chain** – Schickt den bereinigten Feed mit Prompt ans Modell. Der Output Parser ist direkt angebunden – kein Freitext, sondern valides JSON.
- **🤖 Google Gemini Chat Model** – Das angebundene Sprachmodell. Austauschbar gegen andere Modelle ohne den Rest des Workflows zu ändern.
- **📐 Structured Output Parser** – Definiert das JSON-Schema für die LLM-Antwort. Garantiert eine einheitliche, weiterverarbeitbare Struktur.

