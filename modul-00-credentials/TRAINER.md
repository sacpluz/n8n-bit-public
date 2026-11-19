# Modul 0 – Credentials in einem Rutsch anlegen – Erklärungen

Was der Trainer am Beamer erklärt. Die Aufgaben stehen in der [README.md](README.md).

---

## ℹ️ Hintergrund

- Die Public API kann Credentials nur **anlegen** und **löschen**, nicht ändern. Zum Aktualisieren löschen und neu anlegen. Die ID ändert sich dabei, Workflows müssen dann neu zugeordnet werden.
- Alternative ohne Workflow, für Self-Hosted: `n8n import:credentials --input=creds.json`.
