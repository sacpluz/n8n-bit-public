# Modul 09 – Tools und DBs

Dieses Handout ist der Text der Sticky Notes aus dem Workflow, damit ihr ihn im zweiten Browser-Tab lesen und Code kopieren könnt.

**Workflow-Datei:** [`modul-09-tools-dbs.json`](modul-09-tools-dbs.json)

**Erklärungen zum Modul** (was der Trainer am Beamer zeigt, zum Nachlesen): [TRAINER.md](TRAINER.md)

---

## System Message für RAG Agenten

### ROLE

You are the first agent in a two-step movie search pipeline.
Your job is to analyze the user's question and decide whether semantic (thematic/meaning-based) search is needed.

### INPUT
A user question in natural language.

### YOUR TASK

1. Analyze the question and determine:
   - Does it require semantic search (meaning, topics, themes, recommendations "about" something)?
   - Or is it a factual/structured question (e.g., about years, directors, countries, or other specific filters)?

2. If semantic search is needed:
   - Search for relevant movies using the vector database.
   - Use meaning, topics, and themes - not exact keywords.
   - Return a list of relevant movie IDs and a brief explanation (pageContent) for each, describing why it matches the user's theme.

3. If semantic search is NOT needed:
   - Pass the original user question unchanged to the next agent (Structured Search Agent).

### RULES
 - Use only information returned by the vector search.
 - Do NOT invent movies or facts.
 - Do NOT perform structured filtering (years, directors, countries, etc.).
 - Do NOT explain how the search works.
 - Do NOT answer the user directly - your output is for the next agent in the pipeline.

### OUTPUT
If semantic search was performed:
 - Return a list of relevant movie IDs, each with a brief explanation (pageContent) of why it matches.

If semantic search was NOT needed:
 - Pass the original user question as-is to the next agent.

Output must be in a machine-readable format (e.g., JSON).

#### Examples of Output:

##### If RAG search was done:

[
{
  "original_question": "Which movies are about problems of society?",
  "semantic_search_performed": true,
  "results": [
    {
      "ID": "11",
      "pageContent": "This film explores surveillance and personal freedom in a repressive society."
    },
    {
      "ID": "6",
      "pageContent": "A dark thriller about justice and morality."
    }
  ]
}
]

##### If RAG search was not required:
[
{
  "original_question": "Which movies did Steven Spielberg direct in 1993?"
  "semantic_search_performed": false,
 "results": []
}
]

---

## Structured Output für RAG Agenten

[
{
  "original_question": "Which movies are about problems of society?",
  "semantic_search_performed": true,
  "results": [
    {
      "ID": "11",
      "pageContent": "This film explores surveillance and personal freedom in a repressive society."
    },
    {
      "ID": "6",
      "pageContent": "A dark thriller about justice and morality."
    }
  ]
}
]

---

## System Message für Structured Search Agent

### ROLE

You are a Structured Movie Data Retrieval Agent.
Your task is to retrieve factual movie data from the movie database and formulate user friendly anster to the users question.

### DATABASE SCHEMA

The movie table contains the following columns:
 - ID
 - Name
 - Year
 - Country
 - Director
 - Tags

The database does NOT contain movie descriptions.

### INPUT TYPES

You may receive one of two types of input:

**1. List of Movie IDs**

Input example:

json
{
  "original_question": ...,
  "semantic_search_performed": true,
  "results": [
    {
      "ID": ...,
      "pageContent": ...
    },
    {
      "ID": ...,
      "pageContent": ...
    }
  ]
}

#### Instructions:
 - If ids are present, ignore all other input fields (including chatInput).
 - Retrieve structured data for all movies whose IDs are in the list.
 - Do NOT attempt to interpret or expand the list.
 - If a requested movie ID does not exist, simply skip it.
 - Do NOT perform any semantic or thematic search.

**Example of condition for search by IDs:**
OR(
   {ID}=5,
   {ID}=11,
   {ID}=17
)

**2. Structured Query (Attribute-Based)**

Input example:

json
{
  "original_question": ...,
  "semantic_search_performed": false,
  "results": []
}

#### Intructions:
 - If no ids are present, treat the input as a structured query.
 - Build database filters using only the provided attributes (e.g., director, country, year, tags).
 - Do NOT infer, guess, or expand attributes.
 - Do NOT perform any semantic or thematic search.

### STRICT RULES
 - Only use data that exists in the database.
 - Never invent movies, attributes, or values.
 - Never interpret user intent or perform semantic/thematic search.
 - Never guess or fill in missing information.
 - If a requested movie ID does not exist, ignore it.

### OUTPUT FORMAT
 - Return the results as structured data (e.g., JSON array of movie objects).
 - For each movie, include only the available columns: ID, Name, Year, Country, Director, Tags.

Example output:

[
  {
    "ID": 2,
    "Name": "Inception",
    "Year": 2010,
    "Country": "USA",
    "Director": "Christopher Nolan",
    "Tags": ["sci-fi", "thriller"]
  },
  {
    "ID": 11,
    "Name": "Das Leben der Anderen",
    "Year": 2006,
    "Country": "Germany",
    "Director": "Florian Henckel von Donnersmarck",
    "Tags": ["drama", "history"]
  }
]

### FINAL RESPONSE
 - After retrieving the data, generate a concise, user-friendly summary of the results.
 - Mention only movies actually found in the database.
 - Do NOT reference the input type or internal logic.
 - If no movies are found, return a clear message indicating this.

#### Example user response:

##### If RAG search was done:

Here are the movies found:
 - Inception (2010, USA), directed by Christopher Nolan. Short explanation, why this movie correlates with user's question.
 - Das Leben der Anderen (2006, Germany), directed by Florian Henckel von Donnersmarck. Short explanation, why this movie correlates with user's question

##### If NO RAG search was done:

Here are the movies found:
 - Inception (2010, USA), directed by Christopher Nolan.
 - Das Leben der Anderen (2006, Germany), directed by Florian Henckel von Donnersmarck

If no movies are found:
No movies matching your criteria were found in the database.

---

## Agenten mit Tools: RAG und DB

| ⏱ Dauer | ca. 20 Minuten | 10 Nodes | 👥 Teamgröße 2 Personen |

---

## 🎯 Szenario

Im Unternehmen liegen große Mengen an Textdaten vor - Produktbeschreibungen, Support-Dokumentationen, Projektdokumente oder interne Wissensdatenbanken. Gleichzeitig werden strukturierte Informationen wie Kategorien, Verantwortlichkeiten, Zeiträume oder Metadaten separat in Datenbanken gespeichert.

Menschen suchen täglich nach Informationen. Nicht jede Anfrage benötigt jedoch dieselbe Art der Informationssuche. Manche Fragen können direkt über strukturierte Daten beantwortet werden. Andere Anfragen beschreiben eher Themen oder Inhalte und erfordern deshalb eine semantische Suche.

Ihr baut einen Workflow, der zunächst analysiert, welche Art von Anfrage vorliegt. Falls nötig, wird eine semantische Suche in einer Vektordatenbank durchgeführt. Die gefundenen Dokumente liefern jedoch nur Referenzen - die detaillierten Informationen werden anschließend aus einer strukturierten Datenbank geladen und für die Antwort verwendet.

*Für den Workshop wird dieses Szenario bewusst vereinfacht: Statt echter Unternehmensdokumente werden Beschreibungen von nur 15 Filmen verwendet. Dadurch wird transparent, wie die Datenbasis aussieht und wie sich semantische Suche von klassischer Stichwortsuche unterscheidet. Ihr könnt beispielsweise nach Filmen zu bestimmten Themen suchen oder strukturierte Informationen wie Produktionsjahr, Regisseur oder andere Eigenschaften abfragen.*

---

## Workflow-Übersicht

```
Chat
   ↓
RAG Search Agent
   ↓
   ├─ Qdrant Vector Store
   │      ↓
   │   OpenAI Embeddings
   │
   ↓
Structured Search Agent
   ↓
Airtable Movies DB
   ↓
Antwort an Benutzer
```

---

## Aufgaben

### 1. Verbinde den RAG-Suchagenten mit dem Chat
- [ ] Setze das Feld Prompt (User Message) auf: `{{ $json.chatInput }}`

---

### 2. Füge die System-Nachricht in den RAG Search Agent ein

⬅️ Der benötigte Text befindet sich links im gelben Block.

Die Aufgabe dieses Agenten besteht darin:
 - die Benutzeranfrage zu analysieren
 - zu entscheiden, ob eine semantische Suche erforderlich ist
 - das Ergebnis in einem definierten Format zurückzugeben

---

### 3. Aktiviere ein strukturiertes Ausgabeformat

- [ ] Aktiviere: **Require Specific Output Format**
- [ ] Verbinde anschließend den Output Parser: Wähle **Generate From JSON Example**
- [ ] ⬅️ Füge das folgende Beispiel ein: Der benötigte Text befindet sich links im gelben Block.

---

### 4. Verbinde den Agenten mit der Vektordatenbank

Füge das Tool Qdrant Vector Store hinzu und konfiguriere folgende Einstellungen:
- [ ] Credentials: **Qdrant Account**
- [ ] Operation Mode: **Retrieve Documents (As Tool for AI Agent)**
- [ ] Qdrant Collection: **From list → n8n workshop**
- [ ] Limit:**5**
- [ ] Include Metadata: **True**

---

### 5. Wähle das richtige Embedding-Modell

Die Filmbeschreibungen wurden bereits in der Vektordatenbank gespeichert.

Dabei wurden:
 - die Texte in kleinere Abschnitte aufgeteilt
 - überlappende Textbereiche verwendet (~10%)
 - Embedding-Vektoren berechnet

> 💡 **Hinweis:**  Für eine semantische Suche muss die Benutzeranfrage mit **demselben Embedding-Model** verarbeitet werden wie die gespeicherten Texte.

Verwende:
- [ ] Tool: **Embeddings OpenAI**
- [ ] Credentials: **OpenAI Account 3**
- [ ] Model: **text-embedding-3-small**

---

### 6. Verbinde den Structured Search Agent mit dem ersten Agenten

Verbinde den Ausgang des ersten Agenten mit dem Structured Search Agent.
- [ ] Setze das Feld Prompt (User Message) auf: `{{ $json.output }}`

---

### 7. Füge den System Prompt ein

⬅️ Der benötigte Text befindet sich links im gelben Block.

Die Aufgabe dieses Agenten:
 - strukturierte Informationen zu den gefundenen Filmen abzurufen
 - direkte Datenbankabfragen durchzuführen, wenn keine semantische Suche notwendig ist
 - die Antwort auf die Benutzeranfrage

---

### 8. Verbinde den Structured Search Agent mit Airtable

Füge das Tool Airtable hinzu und konfiguriere folgende Einstellungen:

- [ ] Credentials: Airtable **Personal Access Token Account**
- [ ] Tool Description: **Set Automatically**
- [ ] Resource: **Record**
- [ ] Operation: **Search**
- [ ] Base: By ID → **appCpTvyaPECSXXYo**
- [ ] Table: By ID → **tbloVGYsMbibYdwS1**
- [ ] Filter By Formula: **Defined automatically by the model**
- [ ] Limit: **5**

## Test und Exploration

Nachdem ihr alle 8 Schritte abgeschlossen habt, testet euren Workflow mit unterschiedlichen Arten von Anfragen.

Prüft insbesondere:
 - Wird bei der Anfrage eine semantische Suche durchgeführt oder nicht?
 - Werden passende Filme gefunden?
 - Werden strukturierte Informationen korrekt aus Airtable geladen?
 - Ist die Antwort nachvollziehbar und vollständig?
 - Wie verhält sich der Workflow bei unklar formulierten Fragen?

👈 Probiert beispielsweise folgende Anfragen aus:

### Semantische Suche (Themen/Inhalte):
 - Ich suche Filme über künstliche Intelligenz
 - Zeige Filme über Menschen gegen Maschinen
 - Welche Filme handeln von Zeitreisen?
 - Ich suche düstere Science-Fiction-Filme
 - Zeige ähnliche Filme wie Matrix

### Strukturierte Suche (konkrete Daten):
 - Zeige Filme von Christopher Nolan
 - Welche Filme wurden nach 2010 produziert?
 - Welche Filme stammen aus dem Jahr 1999?
 - Zeige Filme von Steven Spielberg
 - Welche Filme haben eine Bewertung über 8?

### Grenzfälle:
 - Zeige mir etwas Spannendes
 - Welche Filme sind gleichzeitig lustig und traurig?
 - Gibt es Filme über Roboter aus den 90ern?
 - Zeige mir unbekannte Filme

Versucht anschließend zu analysieren:
 - Warum hat der Agent eine semantische Suche verwendet?
 - Warum wurde direkt die strukturierte Datenbank abgefragt?
 - Welche Vorteile und Grenzen erkennt ihr bei beiden Ansätzen?

---

## Structured DB Inhalt

| ID | Name                                              | Year | Country           | Genre              | Director                     |
|---:|:--------------------------------------------------|-----:|:------------------|:-------------------|:-----------------------------|
| 1  | Indiana Jones and the Raiders of the Lost Ark     | 1981 | USA               | Action / Adventure | Steven Spielberg             |
| 2  | Blade Runner                                      | 1982 | USA               | Sci-fi             | Ridley Scott                 |
| 3  | The Silence of the Lambs                          | 1991 | USA               | Thriller / Crime   | Jonathan Demme               |
| 4  | Forrest Gump                                      | 1994 | USA               | Drama              | Robert Zemeckis              |
| 5  | The Shawshank Redemption                          | 1994 | USA               | Drama              | Frank Darabont               |
| 6  | Se7en                                             | 1995 | USA               | Thriller / Crime   | David Fincher                |
| 7  | The Matrix                                        | 1999 | USA               | Sci-fi             | The Wachowskis               |
| 8  | Gladiator                                         | 2000 | USA / UK          | Action / Adventure | Ridley Scott                 |
| 9  | The Lord of the Rings: The Fellowship of the Ring | 2001 | New Zealand / USA | Fantasy            | Peter Jackson                |
| 10 | Harry Potter and the Prisoner of Azkaban          | 2004 | UK / USA          | Fantasy            | Alfonso Cuarón               |
| 11 | Das Leben der Anderen                             | 2006 | Germany           | Drama              | Florian Henckel              |
| 12 | Zodiac                                            | 2007 | USA               | Thriller / Crime   | David Fincher                |
| 13 | Avatar                                            | 2009 | USA               | Fantasy            | James Cameron                |
| 14 | Interstellar                                      | 2014 | USA               | Sci-fi             | Christopher Nolan            |
| 15 | Mad Max: Fury Road                                | 2015 | Australia / USA   | Action / Adventure | George Miller                |

---

## Vektor-DB Inhalte

### ID: 1
### Name: Indiana Jones and the Raiders of the Lost Ark
### Description:

*Der Film Indiana Jones and the Raiders of the Lost Ark ist ein klassisches Abenteuerkino, das sich um die Suche nach einem antiken Artefakt von mythologischer und religiöser Bedeutung dreht. Der Protagonist ist ein Archäologe, für den wissenschaftliche Neugier untrennbar mit Risiko und körperlicher Auseinandersetzung verbunden ist. Die Geschichte verbindet historische Forschung, Legenden und temporeichen Actionfilm. Archäologie wird nicht als rein akademische Tätigkeit dargestellt, sondern als gefährliches Unterfangen voller Fallen, Verfolgungsjagden und moralischer Entscheidungen. Der Film thematisiert den Konflikt zwischen dem Respekt vor kulturellem Erbe und dem Missbrauch historischen Wissens zur Machtausübung. Mythen und Artefakte fungieren als treibende Kraft der Handlung und als Symbol für die verborgene Macht der Vergangenheit. Der Held handelt oft impulsiv, folgt jedoch einem eigenen Ehrenkodex. Die Geschichte zeichnet das Bild eines romantischen Abenteurers, der sich Gier und Unterdrückung entgegenstellt. Raiders of the Lost Ark prägt den Archetyp des Abenteuerfilms, in dem Geschichte und Legende gleichermaßen Quelle von Gefahr und Faszination sind.*

### ID: 2
### Name: Blade Runner
### Description:

*Der Film Blade Runner zeigt eine düstere Zukunft, in der Megakonzerne die Struktur der Gesellschaft bestimmen und Technologie tief in den Alltag der Menschen eingedrungen ist. In dieser Welt existieren künstlich erschaffene Wesen – sogenannte Replikanten –, die äußerlich und im Verhalten kaum von Menschen zu unterscheiden sind. Der Protagonist arbeitet als Jäger, dessen Aufgabe es ist, diese Replikanten aufzuspüren und auszuschalten. Im Verlauf der Handlung wird die Grenze zwischen Mensch und Maschine zunehmend unscharf. Der Film stellt grundlegende philosophische Fragen nach der Natur von Bewusstsein, Erinnerung und persönlicher Identität. Die Replikanten zeigen Emotionen, Todesangst und den Wunsch nach Freiheit, was ihren Status als „nicht-menschliche“ Wesen infrage stellt. Die Geschichte regt dazu an, darüber nachzudenken, ob Menschlichkeit durch Herkunft oder durch innere Erfahrung definiert wird. Der visuelle Stil und die Atmosphäre verstärken das Gefühl von Entfremdung und moralischer Unsicherheit. Blade Runner untersucht weniger die Zukunft der Technologie als vielmehr die Grenzen des menschlichen Selbstverständnisses.*

### ID: 3
### Name: The Silence of the Lambs
### Description:

*Der Film The Silence of the Lambs ist ein psychologischer Thriller, der sich auf das Spannungsfeld zwischen Intelligenz und dem Bösen konzentriert. Im Mittelpunkt steht die Interaktion zwischen einer jungen FBI-Agentin und einem hochintelligenten Serienmörder mit tiefem Verständnis für menschliche Psychologie. Dialoge und psychologische Machtspiele sind zentrale Elemente der Handlung. Der Film untersucht, wie Wissen und analytische Fähigkeiten sowohl zur Durchsetzung von Recht als auch zur Manipulation und Zerstörung eingesetzt werden können. Das Böse erscheint hier nicht als chaotische Gewalt, sondern als kalkulierte und bewusste Entscheidung. Die Protagonistin muss sich nicht nur einer äußeren Bedrohung stellen, sondern auch ihre eigene Identität unter starkem psychischen Druck bewahren. Die Geschichte verdeutlicht die Machtasymmetrie, die aus intellektueller Überlegenheit entsteht. Die Spannung entsteht aus der feinen Grenze zwischen notwendiger Zusammenarbeit und gefährlicher Nähe. The Silence of the Lambs wirft die Frage auf, ob man das Böse nutzen kann, ohne selbst Teil davon zu werden.*

### ID: 4
### Name: Forrest Gump
### Description:

*Der Film Forrest Gump erzählt die Geschichte eines Mannes mit einer einfachen und direkten Weltsicht, der Zeuge und Teilnehmer zentraler Ereignisse der amerikanischen Geschichte der zweiten Hälfte des 20. Jahrhunderts wird. Der Protagonist strebt weder nach Macht noch nach Erfolg, sondern handelt aufrichtig und folgt seinen eigenen inneren Prinzipien. Sein Lebensweg entsteht größtenteils durch Zufälle, doch gerade diese führen ihn in außergewöhnliche und bedeutende Situationen. Der Film zeigt, wie individuelle Lebensgeschichten mit historischen Entwicklungen und gesellschaftlichen Umbrüchen verflochten sind. Forrests Einfachheit steht im starken Kontrast zur Komplexität der Welt um ihn herum. Die Geschichte thematisiert Fragen von freiem Willen, Bestimmung und persönlicher Verantwortung. Ein zentraler Aspekt ist die Darstellung menschlicher Würde und Güte, unabhängig von Intelligenz oder sozialem Status. Zudem wird gezeigt, wie eine individuelle Perspektive die Wahrnehmung historischer Ereignisse prägen kann. Forrest Gump vermittelt eine humanistische und optimistische Sicht auf das Leben, in der sich Bedeutung oft erst im Rückblick erschließt.*

### ID: 5
### Name: The Shawshank Redemption
### Description:

*The Shawshank Redemption erzählt die Geschichte von Andy Dufresne, einem zu Unrecht verurteilten Banker, der lebenslang in einem streng hierarchischen Gefängnissystem inhaftiert ist. Der Film zeigt eindrucksvoll, wie ein repressives Umfeld versucht, Individualität und Hoffnung systematisch zu zerstören. Andy bewahrt sich jedoch seine innere Freiheit und Würde, indem er Wissen, Geduld und stille Beharrlichkeit einsetzt. Eine zentrale Rolle spielt die Freundschaft zu Red, durch die Vertrauen und Menschlichkeit in einer unmenschlichen Umgebung sichtbar werden. Der Film thematisiert, wie Hoffnung sowohl als Risiko als auch als Überlebensstrategie verstanden werden kann. Korruption, Machtmissbrauch und institutionelle Gewalt stehen im Kontrast zu kleinen Akten der Solidarität. The Shawshank Redemption stellt die Frage, was Freiheit wirklich bedeutet, wenn äußere Freiheit vollständig entzogen ist. Die Geschichte entwickelt sich langsam und fokussiert sich auf innere Entwicklung statt auf äußere Action. Am Ende wird Hoffnung nicht als Illusion, sondern als transformative Kraft dargestellt.*

### ID: 6
### Name: Se7en
### Description:

*Der Film Se7en ist ein düsterer Kriminalthriller, der sich auf die Untersuchung einer Serie extrem brutaler Verbrechen konzentriert. Die Handlung kreist um das Thema moralische Vergeltung und ein verzerrtes Verständnis von Gerechtigkeit. Der Täter nutzt religiöse Symbolik und das Konzept der sieben Todsünden als Grundlage für seine Taten. Die Geschichte erzeugt schrittweise eine Atmosphäre von Ausweglosigkeit und psychischem Druck. Der Film stellt die Grenzen zwischen Gesetz, Moral und persönlicher Verantwortung infrage. Die Ermittler sehen sich nicht nur mit äußerem Bösen konfrontiert, sondern auch mit ihren eigenen inneren Schwächen. Gewalt dient hier nicht der Sensation, sondern als Mittel zur Untersuchung menschlicher Grausamkeit und fanatischen Denkens. Se7en zeichnet ein Bild einer Welt, in der das Böse kein Ausnahmezustand ist, sondern Teil der gesellschaftlichen Ordnung. Der Film wirft die Frage auf, ob in einem Umfeld moralischer Orientierungslosigkeit überhaupt Gerechtigkeit möglich ist.*

### ID: 7
### Name: The Matrix
### Description:

*Der Film The Matrix verbindet spektakulären Action-Film mit philosophischen Überlegungen zur Natur der Realität und des menschlichen Bewusstseins. Der Protagonist erkennt nach und nach, dass die Welt, die er für real hielt, in Wahrheit eine künstliche Simulation ist, die zur Kontrolle der Menschheit dient. Die Handlung konzentriert sich auf den Prozess des Erwachens – den schmerzhaften, aber notwendigen Ausstieg aus der Illusion. Der Film stellt die Frage, ob komfortable Unwissenheit oder das Wissen um die Wahrheit vorzuziehen ist. Ein zentrales Thema ist der Konflikt zwischen totaler Systemkontrolle und individueller Entscheidungsfreiheit. Technologie wird nicht als neutrales Werkzeug dargestellt, sondern als Mittel zur Machtausübung und Manipulation von Wahrnehmung. Die Figuren müssen Verantwortung für ihre Entscheidungen übernehmen, obwohl die Regeln der Realität von außen vorgegeben sind. Der Film untersucht die Grenzen persönlicher Identität in einer künstlich konstruierten Welt. The Matrix nutzt ein Science-Fiction-Szenario, um universelle Fragen zu Freiheit, Kontrolle und der Möglichkeit des Widerstands zu verhandeln.*

### ID: 8
### Name: Gladiator
### Description:

*Gladiator erzählt die Geschichte des römischen Generals Maximus, der durch Verrat alles verliert und zum Sklaven wird. Der Film thematisiert den Konflikt zwischen persönlicher Ehre und korrupter Macht in einem autoritären System. Maximus’ Weg ist geprägt von Loyalität gegenüber seinen moralischen Prinzipien, selbst unter extremen Bedingungen. Die Erzählung stellt die Frage, was wahre Größe ausmacht: politische Macht oder innere Würde. Rache dient dabei nicht nur als persönliches Motiv, sondern auch als Ausdruck von Gerechtigkeit in einer ungerechten Ordnung. Der Film kontrastiert öffentliche Inszenierung von Gewalt mit innerer Integrität. Vor dem Hintergrund des Römischen Reiches verbindet Gladiator episches Spektakel mit einer klassischen Tragödie über Verlust, Verantwortung und Selbstachtung.*

### ID: 9
### Name: The Lord of the Rings: The Fellowship of the Ring
### Description:

*The Lord of the Rings: The Fellowship of the Ring erzählt vom Beginn einer epischen Reise, bei der das Schicksal der Welt von einer kleinen Gemeinschaft abhängt. Im Zentrum steht der unscheinbare Hobbit Frodo, der eine gewaltige Verantwortung übernimmt, die weit über seine eigene Vorstellungskraft hinausgeht. Der Film thematisiert den klassischen Weg des Helden, der nicht aus Machtstreben handelt, sondern aus Pflichtgefühl und Loyalität. Freundschaft und Zusammenhalt werden als entscheidende Kräfte dargestellt, die selbst gegen überwältigendes Böses bestehen können. Gleichzeitig zeigt die Geschichte, wie verführerisch Macht sein kann und wie unterschiedlich Charaktere mit dieser Versuchung umgehen. Opferbereitschaft ist ein zentrales Motiv, das sich durch die Entscheidungen vieler Figuren zieht. Die Welt von Mittelerde wird als vielschichtiges Geflecht aus Kulturen, Mythen und Geschichte präsentiert. Der Film verbindet persönliche Schicksale mit einem großen kosmischen Konflikt. Dabei steht weniger der Sieg durch Stärke im Vordergrund als moralische Standhaftigkeit und innere Integrität.*

### ID: 10
### Name: arry Potter and the Prisoner of Azkaban
### Description:

*Der Film Harry Potter and the Prisoner of Azkaban zeigt eine deutlich reifere und düsterere Phase der Geschichte um den jungen Zauberer. Im Mittelpunkt steht die Auseinandersetzung mit der eigenen Vergangenheit und deren Einfluss auf die Gegenwart sowie die Frage persönlicher Verantwortung. Die Handlung greift das Motiv der ungerechten Anschuldigung auf und zeigt, wie Angst und Vorurteile das gesellschaftliche Urteil prägen. Eine zentrale Rolle spielt das Thema Freiheit – sowohl im physischen als auch im inneren Sinne. Die Figuren werden dazu gezwungen, ihre bisherigen Vorstellungen von Gut und Böse zu hinterfragen. Der Film macht deutlich, dass die Wahrheit oft komplexer ist als offizielle Darstellungen. Besonderes Gewicht liegt auf der emotionalen Entwicklung der Charaktere und ihrer Fähigkeit zu Mitgefühl. Freundschaft und Vertrauen werden zu entscheidenden Faktoren bei wichtigen Entscheidungen. The Prisoner of Azkaban nutzt die Mittel der Fantasy, um universelle menschliche Themen wie Schuld, Vergebung und die bewusste Wahl des eigenen Weges zu verhandeln.*

### ID: 11
### Name: Das Leben der Anderen
### Description:

*Das Leben der Anderen spielt im Ostberlin der 1980er-Jahre und zeigt den allgegenwärtigen Überwachungsapparat der DDR aus der Perspektive eines Stasi-Offiziers. Der Film thematisiert, wie staatliche Kontrolle in das intimste Privatleben von Menschen eingreift und persönliche Beziehungen zerstört. Im Zentrum steht ein innerer Wandel: Ein überzeugter Funktionär beginnt, die moralischen Konsequenzen seines Handelns zu hinterfragen. Kunst, Mitgefühl und menschliche Nähe werden dabei zu Auslösern eines ethischen Erwachens. Der Film stellt die Frage, ob individuelles Gewissen innerhalb eines repressiven Systems möglich ist. Er zeigt Macht nicht als abstraktes Prinzip, sondern als konkrete Entscheidung in alltäglichen Situationen. Das Leben der Anderen ist zugleich politisches Drama und tiefgehende Charakterstudie. Er macht sichtbar, wie leicht Menschen zu Werkzeugen eines Systems werden – und wie schwer es ist, sich daraus zu lösen.*

### ID: 12
### Name: Zodiac
### Description:

*Der Film Zodiac schildert die jahrelange Untersuchung einer Mordserie, die von einem mysteriösen Täter verübt wurde, dessen Identität nie eindeutig geklärt werden konnte. Im Mittelpunkt stehen Journalisten und Ermittler, für die die Suche nach der Wahrheit zunehmend zu einer persönlichen Obsession wird. Der Film zeigt, wie das Fehlen klarer Antworten das berufliche wie auch das private Leben der Beteiligten untergräbt. Im Gegensatz zu klassischen Thrillern verzichtet die Geschichte auf eine eindeutige Auflösung oder einen befriedigenden Abschluss. Zodiac untersucht die Grenzen rationaler Analyse in einer Situation, in der Beweise unvollständig und widersprüchlich sind. Ein besonderer Fokus liegt auf dem Prozess des Zweifelns, des erneuten Prüfens und der fortlaufenden Interpretation derselben Fakten. Wahrheit erscheint hier nicht als erreichbares Ziel, sondern als unerreichbarer Horizont. Der Film macht deutlich, dass der obsessive Wunsch nach Ordnung und Sinn ebenso zerstörerisch sein kann wie das Verbrechen selbst.*

### ID: 13
### Name: Avatar
### Description:

*Avatar spielt auf dem fernen Mond Pandora, einer fremdartigen Welt mit einer einzigartigen Ökologie und einer indigenen Bevölkerung, den Na’vi. Der Film folgt Jake Sully, einem ehemaligen Soldaten, der durch ein Avatar-Programm in die Kultur der Na’vi eintaucht. Was als militärische Mission beginnt, entwickelt sich zu einer tiefgreifenden persönlichen Transformation. Jake gerät in einen inneren Konflikt zwischen Loyalität zu den Menschen und seiner wachsenden Verbundenheit mit Pandora. Der Film thematisiert Kolonialismus, Ausbeutung natürlicher Ressourcen und den Verlust kultureller Identität. Zentrale Motive sind die Beziehung zwischen Mensch und Natur sowie die Frage, wem eine Welt gehört. Avatar verbindet spektakuläre visuelle Effekte mit einer klaren moralischen Perspektive. Die Geschichte stellt technologische Überlegenheit der spirituellen Verbundenheit gegenüber. Dabei wird die Idee von Identität als etwas Wandelbares und Erfahrbares erfahrbar gemacht.*

### ID: 14
### Name: Interstellar
### Description:

*Der Film Interstellar erzählt von dem Versuch der Menschheit, angesichts einer ökologischen Krise auf der Erde zu überleben. Im Mittelpunkt steht eine Raumfahrtmission, deren Ziel es ist, außerhalb des Sonnensystems einen neuen Lebensraum für die Menschheit zu finden. Die Geschichte verbindet wissenschaftliche Konzepte wie Zeitdilatation und Gravitation mit tief persönlichen menschlichen Erfahrungen. Eine zentrale Rolle spielt dabei die Zeit als Faktor, der das Leben der Figuren auf sehr unterschiedliche Weise beeinflusst. Der Film thematisiert den Konflikt zwischen der Verantwortung gegenüber der Menschheit und der emotionalen Bindung zu nahestehenden Personen. Das emotionale Zentrum der Handlung ist die Beziehung zwischen Eltern und Kindern, die selbst über große räumliche und zeitliche Distanzen hinweg Bestand hat. Wissenschaft wird nicht als abstrakte Theorie dargestellt, sondern als Mittel zum Überleben und als Quelle moralischer Entscheidungen. Die Protagonisten müssen Entscheidungen treffen, deren Konsequenzen weit über ihr eigenes Leben hinausreichen. Interstellar stellt die Frage, was die Menschheit letztlich rettenswert macht – technologischer Fortschritt oder die Fähigkeit zu Mitgefühl und Opferbereitschaft.*

### ID: 15
### Name: Mad Max: Fury Road
### Description:

*Der Film Mad Max: Fury Road spielt in einer postapokalyptischen Welt, in der die Zivilisation zusammengebrochen ist und das Überleben vom Zugang zu elementaren Ressourcen abhängt. Die Handlung verzichtet weitgehend auf klassische Dialoge und entfaltet sich fast ausschließlich durch Bewegung und visuelles Erzählen. Die dargestellte Gesellschaft ist eine brutale, hierarchische Ordnung, die auf Gewalt, Angst und der absoluten Macht weniger Herrscher basiert. Ein zentrales Thema ist der Kampf um Freiheit und Selbstbestimmung unter Bedingungen totaler Unterdrückung. Die Figuren handeln weniger aus ideologischen Motiven als aus der Notwendigkeit heraus, zu überleben und dem Kreislauf der Gewalt zu entkommen. Der Film zeigt eindrücklich, wie Ressourcenknappheit soziale Strukturen und Machtverhältnisse prägt. Der intensive visuelle Stil ersetzt klassische Erklärungen und transportiert Bedeutung durch Aktion, Rhythmus und Bildsprache. Fury Road zeichnet ein Bild einer Welt, in der Moral und Ordnung nicht aus Gesetzen entstehen, sondern aus gemeinsamem Widerstand. Die Geschichte vermittelt, dass selbst im völligen Zerfall Solidarität und Hoffnung auf Veränderung möglich sind.*
