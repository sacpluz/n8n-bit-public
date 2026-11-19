# Modul 9 – Agenten mit Tools: Vektorsuche und Datenbank

> **Nicht Teil des Workshops.** Dieses Modul ist im Workshop out of scope. Ihr könnt es danach im eigenen Tempo durcharbeiten.

| ⏱ Dauer | 30 Minuten | 8 Aufgaben | 👥 Teams à 2 Personen |
|---|---|---|---|

**Workflow-Datei:** [`modul-09-tools-dbs-challenge.json`](modul-09-tools-dbs-challenge.json)

**Erklärungen zum Modul** (was der Trainer am Beamer zeigt, zum Nachlesen): [TRAINER.md](TRAINER.md)

> 💡 Handout im **zweiten Browser-Tab** offen lassen. In diesem Modul kopiert ihr System Messages und JSON von hier in den Workflow.

---

## 🎯 Szenario

Im Unternehmen liegt Wissen an zwei Stellen: lange Texte (Dokumentationen, Beschreibungen, Tickets) und strukturierte Daten (Kategorien, Jahre, Verantwortliche) in einer Datenbank. Manche Fragen lassen sich nur über den **Inhalt** beantworten („Welche Projekte hatten Probleme mit Datenschutz?“), andere nur über **Fakten** („Welche Projekte leitet Frau Weber?“).

Ihr baut eine Suche aus zwei Agenten:

1. Der **RAG Search Agent** entscheidet, ob eine Themensuche nötig ist, und sucht dann in der **Vektordatenbank** (Qdrant).
2. Ein **IF** schickt das Ergebnis auf einen von zwei Wegen.
3. Der **Structured Search Agent** holt die Fakten aus der **Datenbank** (Airtable) und formuliert die Antwort.

Statt Unternehmensdaten nehmen wir **15 Filme**. So seht ihr genau, was in den Datenbanken steht (siehe [Anhang](#anhang-inhalt-der-datenbanken)).

```
Start ─ AUFGABE 0 ─ Testfragen ─┐
                                ├─ RAG Search Agent ─ IF ─┬─ true:  Prompt – IDs nachschlagen ──┐
Chat – eigene Fragen ───────────┘     │ Qdrant + Embeddings └─ false: Prompt – Frage weitergeben ┴─ Structured Search Agent ─ Abgabe oder Chat?
                                                                                                       │ Airtable
```

**Input für die Abgabe (steckt schon im Node `Testfragen`):**

```json
[
  { "chatInput": "Welche Filme handeln von staatlicher Überwachung?" },
  { "chatInput": "Welche Filme hat David Fincher gedreht?" }
]
```

Die erste Frage braucht die Vektorsuche, die zweite nur die Datenbank. Nur wenn beide richtig beantwortet werden, zählt die Abgabe.

⚠️ **Node-Namen nicht ändern.** `Ergebnis prüfen` liest die Nodes per Namen.

**Credentials:** Qdrant, OpenAI, Azure und Airtable sind in den Nodes vorausgewählt und für euch freigegeben. Ihr braucht keine eigenen Keys. Ist ein Feld rot oder leer, sagt dem Trainer Bescheid.

---

## Aufgaben

### 00 · SET – Namen eintragen

| Name | Typ | Wert |
|---|---|---|
| name | String | euer Vor- und Nachname, z. B. `Anna Müller` |

### 01 · AI AGENT – RAG Search Agent

Node `AUFGABE 1 – RAG Search Agent` öffnen.

| Einstellung | Wert |
|---|---|
| Source for Prompt (User Message) | **Define below** |
| Prompt (User Message) | `{{ $json.chatInput }}` |
| Options → Add Option → System Message | Text unten, **alles ersetzen** |

`chatInput` liefern beide Starts: der Node `Testfragen` und der Chat.

```
# ROLE
You are the first agent in a two-step movie search pipeline.
You decide whether the user's question needs a semantic search (topics, themes, moods, content) and, if so, you run it.

# YOUR TASK
1. Read the question.
   - Topic, theme, mood or content ("movies about surveillance", "dark sci-fi", "similar to Matrix") → semantic search.
   - Only facts (year, director, country, genre, name) → no semantic search.
   - Both (e.g. "robot movies from the 90s") → semantic search. The next agent applies the year filter.
2. Semantic search: call the vector store tool. Search for meaning, not for exact words.
   For every relevant hit, return the movie ID from the metadata and one sentence why it matches.
3. No semantic search: return the question and an empty results list.

# RULES
- Use only what the vector store returns. Never invent movies or IDs.
- Do not filter by year, director, country or genre. That is the next agent's job.
- Do not answer the user. Your output goes to the next agent.
- original_question is always the user's question, unchanged.

# OUTPUT
{
  "original_question": "Which movies are about problems of society?",
  "semantic_search_performed": true,
  "results": [
    { "ID": "11", "pageContent": "Surveillance and personal freedom in a repressive state." },
    { "ID": "6", "pageContent": "A dark thriller about justice and morality." }
  ]
}

Without semantic search:
{
  "original_question": "Which movies did David Fincher direct?",
  "semantic_search_performed": false,
  "results": []
}
```

### 02 · STRUCTURED OUTPUT PARSER – festes Antwortformat

Der nächste Schritt (IF) soll **ohne KI** entscheiden. Dafür braucht er ein festes Feld `semantic_search_performed`. Das kennt ihr aus Modul 8.

Im Node `AUFGABE 1 – RAG Search Agent`:

| Einstellung | Wert |
|---|---|
| Require Specific Output Format | **an** |

Unter dem Agenten erscheint der Anschluss **Output Parser**. Zieht ihn auf den Node `AUFGABE 2 – Structured Output Parser`. Dann diesen Node öffnen:

| Einstellung | Wert |
|---|---|
| Schema Type | **Generate From JSON Example** |
| JSON Example | Beispiel unten |

```json
{
  "original_question": "Which movies are about problems of society?",
  "semantic_search_performed": true,
  "results": [
    { "ID": "11", "pageContent": "Surveillance and personal freedom in a repressive state." },
    { "ID": "6", "pageContent": "A dark thriller about justice and morality." }
  ]
}
```

⚠️ Das Beispiel beginnt mit `{`, nicht mit `[`. Ein Array würde den IF in Aufgabe 5 aushebeln.

### 03 · QDRANT VECTOR STORE – Vektorsuche als Tool

Node `AUFGABE 3 – Qdrant Vector Store` (hängt schon als Tool am Agenten).

| Einstellung | Wert |
|---|---|
| Credential to connect with | `Qdrant account` (vorausgewählt) |
| Operation Mode | **Retrieve Documents (As Tool for AI Agent)** |
| Description | Text unten |
| Qdrant Collection | From list → **n8n workshop** |
| Limit | `5` |
| Include Metadata | **an** |

```
Semantic search in the descriptions of 15 movies. Use it for questions about topics, themes, moods or content. Returns matching text passages with the movie ID in the metadata. Does not know years, directors or countries.
```

💡 Die **Description** liest der Agent, um zu entscheiden, wann er das Tool benutzt. Sie ist so wichtig wie die System Message.

### 04 · EMBEDDINGS OPENAI – dasselbe Modell wie beim Speichern

Die Filmbeschreibungen wurden mit `text-embedding-3-small` in Vektoren umgerechnet (in Abschnitte geteilt, ca. 10 % Überlappung). Eure Frage muss mit **demselben Modell** umgerechnet werden, sonst passen die Vektoren nicht zusammen.

| Einstellung | Wert |
|---|---|
| Credential to connect with | `OpenAI account 3` (vorausgewählt) |
| Model | **text-embedding-3-small** |

### 05 · IF – Semantische Suche?

Node `AUFGABE 5 – IF: Semantische Suche?`. Eine Bedingung anlegen:

| Einstellung | Wert |
|---|---|
| Value 1 | `{{ $json.output.semantic_search_performed }}` |
| Operator | **Boolean → is true** |

Die zwei Nodes dahinter sind fertig:

| Ausgang | Node | Was er baut |
|---|---|---|
| true | `Prompt – IDs nachschlagen` | „Lade genau diese IDs: 11, 2, …“ plus die Begründungen |
| false | `Prompt – Frage weitergeben` | „Keine Themensuche, filtere nach den Spalten“ plus die Frage |

`Pfade zusammenführen` (Merge) sammelt beide Wege wieder ein.

### 06 · AI AGENT – Structured Search Agent

Node `AUFGABE 6 – Structured Search Agent`.

| Einstellung | Wert |
|---|---|
| Source for Prompt (User Message) | **Define below** |
| Prompt (User Message) | `{{ $json.prompt }}` |
| Options → Add Option → System Message | Text unten, **alles ersetzen** |

```
# ROLE
You are the second agent in a movie search pipeline. You load facts from the movie database and answer the user's question.

# DATABASE
Table with the columns: ID, Name, Year, Country, Genre, Director.
The table has no descriptions and no ratings.

# INPUT
You get one of two prompts:
1. "Look up exactly these movie IDs" → load these IDs. If the question also names a year, director, country or genre, add that filter.
2. "No semantic search was done" → build a filter from the question.

# AIRTABLE FORMULAS (examples)
- IDs: OR({ID}=6, {ID}=11)
- Director: {Director}='David Fincher'
- Year: {Year}>2010
- Genre contains: FIND('Sci-fi', {Genre})
- Combined: AND(OR({ID}=2, {ID}=7), {Year}<2000)

# RULES
- Mention only movies the tool returned. Never invent movies or values.
- If a filter needs a column that does not exist (e.g. rating), say so.
- If nothing is found, say: "Dazu habe ich in der Datenbank keine Filme gefunden."

# ANSWER
Answer in the language of the question, short, as a list:
- Name (Year, Country), Regie: Director – for semantic hits add one sentence why it matches.
```

### 07 · AIRTABLE – Filmdatenbank als Tool

Node `AUFGABE 7 – Airtable: Movies DB` (hängt schon als Tool am Agenten).

| Einstellung | Wert |
|---|---|
| Credential to connect with | `Airtable Personal Access Token account` (vorausgewählt) |
| Tool Description | **Set Manually**, Text unten |
| Resource | **Record** |
| Operation | **Search** |
| Base | By ID → `appCpTvyaPECSXXYo` |
| Table | By ID → `tbloVGYsMbibYdwS1` |
| Filter By Formula | ✨ **Let the model define this parameter** (Stern-Knopf rechts am Feld) |
| Return All | aus |
| Limit | `15` |

```
Searches the movie table (columns ID, Name, Year, Country, Genre, Director) with an Airtable formula. Use it for facts like year, director, country, genre, or to load movies by ID. Contains no descriptions.
```

### Abgabe

**Execute workflow**. Beide Testfragen laufen durch, `Ergebnis prüfen` zeigt im Feld `hinweise`, was noch fehlt, und trägt euch ins Scoreboard ein.

Kontrolle in `Ergebnis prüfen`:

| Feld | Soll |
|---|---|
| `korrekt` | `✅ Ja` |
| `antworten` | zwei Antworten: eine mit *Das Leben der Anderen*, eine mit *Se7en* und *Zodiac* |

Die Antworten kommen von einem Sprachmodell und sind nicht jedes Mal gleich. Geprüft werden nur die Filmnamen und die Felder aus Aufgabe 2.

---

## 💬 Ausprobieren im Chat

Unten **Open chat** klicken und eigene Fragen stellen. Danach im Tab **Executions** nachsehen, welchen Weg der IF genommen hat und was die Tools zurückgegeben haben.

| Art | Beispiele |
|---|---|
| Thema (Vektorsuche) | Ich suche Filme über künstliche Intelligenz · Welche Filme handeln von Zeitreisen? · Zeig mir Filme wie Matrix · Düstere Science-Fiction |
| Fakten (Datenbank) | Filme von Christopher Nolan · Filme nach 2010 · Filme aus dem Jahr 1994 · Filme aus Deutschland |
| Grenzfälle | Filme über Roboter aus den 90ern · Filme mit Bewertung über 8 · Zeig mir etwas Spannendes · Filme, die lustig und traurig sind |

Fragen zum Nachdenken:

- Warum hat der erste Agent gesucht oder nicht gesucht?
- Bei „Roboter aus den 90ern“: Kommt der Jahresfilter beim zweiten Agenten an?
- „Bewertung über 8“: Was antwortet der Agent, wenn es die Spalte nicht gibt?

---

## 🏆 Wertung

Die schnellste **korrekte** Abgabe gewinnt. Details: [Scoreboard](../scoreboard.md).

## 🔧 Wenn es nicht klappt

| Symptom | Ursache |
|---|---|
| Hinweis „liefert Text statt JSON“ | Require Specific Output Format aus, oder der Output Parser hängt nicht am Agenten |
| Hinweis „JSON-Beispiel ist ein Array“ | Im Output Parser steht `[ { … } ]`. Nur das Objekt `{ … }` einfügen |
| Alle Fragen landen auf false | IF prüft `$json.semantic_search_performed` statt `$json.output.semantic_search_performed` |
| Fehler „Wrong input dimensions“ o. ä. bei Qdrant | Falsches Embedding-Modell. Muss `text-embedding-3-small` sein |
| Vektorsuche liefert Treffer ohne `ID` | Include Metadata aus |
| Agent ruft Qdrant nie auf | Description leer, oder die System Message fehlt |
| Airtable: `INVALID_FILTER_BY_FORMULA` | Das Modell hat eine Spalte erfunden. In der System Message stehen die echten Spalten |
| Antwort „keine Filme gefunden“ bei David Fincher | Filter By Formula ist ein fester Text statt ✨ *defined by the model* |
| Credential-Feld leer oder rot | Credential ist nicht für euch freigegeben. Trainer fragen |
| `429 Too Many Requests` | Zu viele Anfragen gleichzeitig. Eine Minute warten, dann neu starten |
| Chat antwortet nicht | Im Chat-Modus läuft am Ende `Antwort an den Chat`. Wurde der Node gelöscht? |

---

## 🚀 Schon fertig?

1. **Grenzfall reparieren:** Bringt „Filme über Roboter aus den 90ern“ zum richtigen Ergebnis (nur *The Matrix*). Wo geht der Jahresfilter verloren, in der System Message des ersten Agenten oder im Prompt des zweiten?
2. **Ein Agent statt zwei:** Hängt Qdrant und Airtable als Tools an *einen* Agenten und löscht IF und Merge. Was wird einfacher, was unzuverlässiger?
3. **Tool-Beschreibung sabotieren:** Schreibt in die Description von Qdrant „Use this for directors“. Was macht der Agent jetzt bei David Fincher?
4. **Gedächtnis:** Hängt an den Structured Search Agent ein **Simple Memory** und fragt im Chat nach: „Und welcher davon ist der älteste?“

---

## Anhang: Inhalt der Datenbanken

### Airtable (strukturiert)

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

### Qdrant (Beschreibungen, je Film ein Text)

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
### Name: Harry Potter and the Prisoner of Azkaban
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
