# Modul 09 – Tools und DBs – Erklärungen

Was der Trainer am Beamer erklärt. Die Aufgaben stehen in der [README.md](README.md).

---

## Modul 9 und 10 - Node-Übersicht

1. **Chat Trigger **– Startpunkt des Workflows. Empfängt Benutzeranfragen aus dem Chat und startet die Verarbeitung des Workflows. Übergibt die Nachricht als chatInput.
2. **AI Agent (RAG Search Agent)** – Analysiert die Benutzeranfrage und entscheidet, ob eine semantische Suche erforderlich ist. Führt bei Bedarf eine thematische Suche durch und gibt strukturierte Ergebnisse zurück.
3. **Structured Output Parser** – Erzwingt ein definiertes JSON-Format für die Antwort des ersten Agenten. Dadurch wird sichergestellt, dass nachfolgende Komponenten immer dieselbe Datenstruktur erhalten.
4. **Qdrant Vector Store** – Führt eine semantische Suche in der Vektordatenbank durch und liefert relevante Dokumente bzw. Filmreferenzen zurück.
5. **Embeddings OpenAI** – Berechnet Embedding-Vektoren für Benutzeranfragen. Für eine korrekte semantische Suche muss dieselbe Embedding-Methode verwendet werden wie beim Erstellen der Vektordatenbank.
6. **Google Gemini Chat Model** – Sprachmodell, das von beiden Agenten verwendet wird. Führt Analyse-, Entscheidungs- und Antwortgenerierungsschritte aus.
7. **AI Agent (Structured Search Agent)** – Verarbeitet die Ergebnisse des ersten Agenten. Ruft strukturierte Daten aus der Datenbank ab und formuliert eine benutzerfreundliche Antwort.
8. **Airtable Tool (Movies DB)** – Führt Datenbankabfragen auf der Filmdatenbank aus. Die Suchfilter werden dynamisch vom Agenten erstellt.
