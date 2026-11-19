# Modul 9 – Agenten mit Tools: Vektorsuche und Datenbank – Erklärungen

Was der Trainer am Beamer erklärt. Die Aufgaben stehen in der [README.md](README.md).

---

## Voraussetzungen

Das Modul läuft nur, wenn der Trainer vorher Folgendes bereitgestellt hat (Details im Facilitator Guide):

- Qdrant-Cluster mit der befüllten Collection `n8n workshop`
- Airtable-Base mit der Filmtabelle
- Die Credentials für Qdrant, OpenAI, Gemini und Airtable sind in n8n mit allen Teilnehmern **geteilt**. Dann sind sie in der Vorlage vorausgewählt, und niemand braucht eigene Keys.
- Genug Kontingent bei Gemini und OpenAI für alle gleichzeitig

Fehlt etwas davon, zeigt der Trainer das Modul als Demo.

---

## Ablauf am Beamer (10 min) – Playground links im Workflow

Alle Nodes in der gelben Notiz **Playground – Vektorsuche und Datenbank**. Jeweils *Execute step*.

| Schritt | Node | Was zeigen |
|---|---|---|
| 1 | **Playground – Vektorsuche: Überwachung** | Drei Treffer, ganz oben *Das Leben der Anderen*. Im Output `metadata.ID` und `pageContent` zeigen. Das Wort „Überwachung“ muss nicht wörtlich im Text stehen, die Suche vergleicht Bedeutung |
| 2 | **Playground – Vektorsuche: David Fincher** | Es kommen trotzdem drei Treffer, aber beliebige. Die Vektorsuche liefert immer *die ähnlichsten* Texte, auch wenn keiner passt. Regisseure stehen nicht in den Beschreibungen |
| 3 | **Playground – Airtable: Filme von David Fincher** | Formel `{Director}='David Fincher'` → genau *Se7en* und *Zodiac*. Fakten gehören in die Datenbank |
| 4 | **Playground – Airtable: IDs 6 und 11** | Formel `OR({ID}=6, {ID}=11)`. So lädt der zweite Agent die Filme, die die Vektorsuche gefunden hat |
| 5 | **Playground – Embeddings OpenAI** | Hängt an beiden Vektorsuchen. Modell `text-embedding-3-small`, dasselbe wie beim Befüllen von Qdrant. Ein anderes Modell erzeugt Vektoren anderer Länge und Bedeutung |

Danach im Hauptstrang zeigen:

- **AUFGABE 1 und 6** sind zwei Agenten mit je einem Tool. Das Sprachmodell (Gemini) ist für beide fertig angeschlossen.
- **AUFGABE 5 (IF)**: Die Entscheidung „Themensuche ja/nein“ trifft der erste Agent, aber den Weg danach bestimmt ein IF ohne KI. Das geht nur, weil der Output Parser (Aufgabe 2) ein festes Feld `semantic_search_performed` erzwingt. Rückgriff auf Modul 8.
- **Testfragen vs. Chat**: *Execute workflow* ist die Abgabe mit zwei festen Fragen. *Open chat* ist zum Ausprobieren, dann endet der Workflow in `Antwort an den Chat` und nichts geht ans Scoreboard. Das entscheidet `Abgabe oder Chat?`.

## Node-Übersicht

| Node | Aufgabe im Workflow |
|---|---|
| **Start – Execute workflow** / **Testfragen** | Abgabe: zwei feste Fragen als `chatInput` |
| **Chat – eigene Fragen** | Chat-Trigger, liefert die Nachricht ebenfalls als `chatInput` |
| **RAG Search Agent** | Entscheidet, ob eine Themensuche nötig ist, und ruft dafür Qdrant auf |
| **Structured Output Parser** | Erzwingt das JSON-Format `original_question`, `semantic_search_performed`, `results` |
| **Qdrant Vector Store** | Vektorsuche in den 15 Filmbeschreibungen, als Tool für den Agenten |
| **Embeddings OpenAI** | Rechnet die Suchanfrage in einen Vektor um |
| **Google Gemini Chat Model** | Sprachmodell für beide Agenten, Temperatur 0 |
| **IF: Semantische Suche?** | Wählt den Weg: IDs nachschlagen oder Frage direkt weitergeben |
| **Prompt – …** / **Pfade zusammenführen** | Bauen den Prompt für den zweiten Agenten, Merge sammelt beide Wege ein |
| **Structured Search Agent** | Baut eine Airtable-Formel, lädt die Fakten und antwortet |
| **Airtable: Movies DB** | Tabelle mit ID, Name, Year, Country, Genre, Director. Die Formel schreibt das Modell (`$fromAI`) |
| **Ergebnis prüfen** / **Scoreboard** | Prüft beide Antworten und trägt das Ergebnis ein |

## Warum zwei Datenbanken?

| | Vektordatenbank (Qdrant) | Strukturierte Datenbank (Airtable) |
|---|---|---|
| Speichert | Text als Zahlenvektor | Felder mit festen Werten |
| Findet | Ähnliche Bedeutung, auch mit anderen Worten | Exakte Treffer per Filter |
| Gut für | „Filme über Schuld und Vergebung“ | „Filme von 1994“ |
| Schlecht für | Jahre, Namen, Zählen | Themen, Stimmungen |
| Liefert immer | die k ähnlichsten Treffer, auch wenn keiner passt | nur, was den Filter erfüllt |

## Was in v01 anders war

- Der zweite Agent musste selbst erkennen, ob IDs kommen oder nicht. Jetzt entscheidet ein IF, und der Prompt sagt dem Agenten genau, was zu tun ist.
- Bei IDs hat der zweite Agent alle anderen Angaben ignoriert. Dadurch ging bei „Roboter aus den 90ern“ der Jahresfilter verloren. Jetzt soll er zusätzliche Filter anwenden (Bonusaufgabe 1 prüft, ob das klappt).
- Das JSON-Beispiel war ein Array, der zweite Agent erwartete ein Objekt. Jetzt ist es ein Objekt.
- Die Tool-Beschreibungen schreiben die Teilnehmer selbst ein, statt „Set Automatically“.
