# Modul 05 – Error Handling – Erklärungen

Was der Trainer am Beamer erklärt. Die Aufgaben stehen in der [README.md](README.md).

---

**Execution Log**

Unter **Executions** kannst du jeden vergangenen Workflow-Run öffnen. Mit **Debug in Editor** werden die exakten Fehler-Daten in den Workflow geladen — ideal zum Debuggen von Edge Cases in Produktion. **Reagenzglas** bedeutet in Testing - Pandon: Published.

---

**DebugHelper – throwError**

Nützlich um den **Error Trigger** Workflow zu testen, ohne auf einen echten Fehler warten zu müssen.

**Oder**, um in einem Workflow mit Testdaten zu arbeiten.

1. DebugHelper ausführen → Workflow crasht
2. Error Trigger Workflow feuert
3. Fehler landet im Log / Gmail

---

# ERROR TRIGGER WORKFLOW *(separater Workflow in n8n)*

#### Startet automatisch wenn ein anderer Workflow crasht. Muss **nicht aktiv** gesetzt sein — der einzige Node in n8n mit dieser Eigenschaft. Setup: Im Haupt-Workflow unter **⋮ → Settings → Error Workflow** diesen Workflow auswählen.

---

**Verfügbare Felder im Error Trigger:**

```
$json.execution.id
$json.execution.url
$json.error.message
$json.error.description
$json.workflow.id
$json.workflow.name
```

**Empfehlung:** Mindestens ein zentrales Error-Logging-Workflow einrichten. Alle anderen Workflows zeigen darauf.
