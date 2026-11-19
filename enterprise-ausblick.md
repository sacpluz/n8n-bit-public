# Enterprise-Ausblick: Vom Workshop in den Betrieb

| ⏱ Dauer | 25 Minuten | Format: Kurzvortrag und Diskussion |
|---|---|---|

Im Workshop baut jede:r allein, mit Testdaten, ohne Folgen. Im Unternehmen ist das anders. Die Rückmeldung aus dem ersten Durchlauf war eindeutig: Für den produktiven Einsatz braucht es einen Rahmen. Dieser Block zeigt die sechs Themen, die man klären muss, bevor der zweite Workflow live geht.

Er liefert Leitfragen, keine fertigen Antworten. Die hängen von Lizenz, Hosting und Organisation ab.

---

## 1. Framework: Regeln, bevor es viele werden

Ohne gemeinsame Regeln sieht jeder Workflow anders aus und niemand findet sich im Workflow der Kollegin zurecht.

- **Namen:** einheitliches Schema für Workflows, Nodes und Credentials (z. B. `Bereich – Zweck – Umgebung`).
- **Bausteine:** wiederkehrende Logik als Sub-Workflow, nicht kopieren.
- **Pflicht-Elemente:** Fehlerbehandlung (Modul 5), eine Sticky Note mit Zweck und Ansprechperson, Logging.
- **Leitplanken:** Welche Nodes sind erlaubt? Wann ist ein Code-Node in Ordnung? Welche KI-Modelle dürfen welche Daten sehen?

**Leitfrage:** Wer legt diese Regeln fest und wer prüft sie?

## 2. Zusammenarbeit

- **Rollen und Projekte:** Wer darf bauen, wer nur ausführen, wer freigeben?
- **Gemeinsames Arbeiten:** Zwei Personen am selben Workflow überschreiben sich gegenseitig. Zuständigkeit pro Workflow klären.
- **Review:** Vier-Augen-Prinzip vor dem Aktivieren.
- **Fachbereich und IT:** Der Fachbereich kennt den Prozess, die IT kennt Daten und Schnittstellen. Wer baut, wer berät?

**Leitfrage:** Baut der Fachbereich selbst (Citizen Development), oder liefert er Anforderungen an ein zentrales Team?

## 3. Day-2-Operation: der Tag nach dem Go-live

- **Überwachung:** Wer merkt, wenn ein Workflow nachts fehlschlägt? Error-Workflow mit Benachrichtigung (Modul 5).
- **Stille Fehler:** Ein Workflow kann grün durchlaufen und trotzdem falsche Daten liefern. Das habt ihr heute selbst erlebt. Fachliche Prüfungen einbauen.
- **Betrieb:** Updates, Backups, Wiederanlauf, Lastgrenzen der Instanz.
- **Zuständigkeit:** Jeder produktive Workflow hat eine verantwortliche Person und eine Vertretung.

**Leitfrage:** Was passiert, wenn die Person, die den Workflow gebaut hat, im Urlaub ist?

## 4. Lifecycle Management

- **Umgebungen:** Entwicklung, Test, Produktion getrennt halten.
- **Versionierung:** Workflows als JSON in Git (so wie dieses Repo). Änderungen werden nachvollziehbar und rückholbar.
- **Deployment:** Wie kommt ein Workflow von Test nach Produktion? Achtung bei Verweisen: IDs von Credentials und Sub-Workflows unterscheiden sich je Umgebung.
- **Stilllegung:** Workflows, die niemand mehr braucht, abschalten und archivieren.

**Leitfrage:** Können wir den Stand von letzter Woche wiederherstellen?

## 5. Credential Handling

- Credentials **zentral** anlegen und gezielt teilen. Keine Zugangsdaten in Code-Nodes, Expressions oder Sticky Notes.
- **Technische Benutzer** statt persönlicher Konten. Sonst steht der Workflow, wenn jemand das Unternehmen verlässt.
- **Kleinste nötige Rechte** pro Credential.
- **Rotation und Ablauf:** OAuth-Tokens laufen ab. Wer erneuert sie?
- Bei größeren Installationen: externer Secret Store.

**Leitfrage:** Wer darf welche Credential benutzen, und wer sieht, wo sie überall im Einsatz ist?

## 6. Data Management

- **Welche Daten fließen wohin?** Personenbezogene Daten, Kundendaten, Geschäftsgeheimnisse.
- **Execution-Daten:** n8n speichert Ein- und Ausgaben jedes Laufs. Aufbewahrungsdauer festlegen, sensible Läufe nicht speichern.
- **KI-Nodes:** Was an ein Modell geschickt wird, verlässt die Instanz. Anbieter, Region und Vertragslage klären.
- **Datenqualität:** Automatisierung verstärkt Fehler in den Quelldaten. Prüfen wie in Modul 1 und 2.
- **Datenkompetenz:** JSON und Data Mapping sind die Grundlage. Wer im Fachbereich weiterbauen soll, braucht dafür ein eigenes Training.

**Leitfrage:** Dürften diese Daten das Unternehmen in dieser Form verlassen?

---

## Diskussion (10 min)

1. Welcher Prozess bei euch wäre ein guter erster Kandidat?
2. Welches der sechs Themen ist bei euch am wenigsten geklärt?
3. Wer müsste mit am Tisch sitzen, bevor ihr startet?

## Empfehlung für den Start

- Mit **einem** Prozess beginnen, der wenig Risiko hat und spürbar Zeit spart.
- Die Regeln aus Abschnitt 1 an diesem ersten Workflow entwickeln, nicht auf Vorrat.
- Früh klären: Zuständigkeit, Credentials, Fehlerbenachrichtigung.
- Fachbereich gezielt in JSON und Data Mapping schulen, bevor er selbst baut.
