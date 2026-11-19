# Modul 05 – Error Handling

Dieses Handout ist der Text der Sticky Notes aus dem Workflow, damit ihr ihn im zweiten Browser-Tab lesen und Code kopieren könnt.

**Workflow-Datei:** [`modul-05-error-handling.json`](modul-05-error-handling.json)

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

---

# 🛠️ Setup Guide  
**Author:** [Nate Herk](https://www.youtube.com/@nateherk)

---
### 1️⃣ Link This as Your Error Workflow  
- Go to **Settings** → **Error Workflow** → Select this workflow as your global error handler.
- The parent workflow must be active.
---

#### 2️⃣ Error Logger (Google Sheets)  
🔗 **[Make a copy of the template](https://docs.google.com/spreadsheets/d/1Aiz1cpv1k6Qlmp1Cjmj0ZtNuG6fC_cG24O_XaiP8NOs/edit?usp=sharing)**  
- Set up your Google Sheets credentials  
- Connect your Google credential  
- Customize the sheet as needed (optional)
---

#### 3️⃣ Slack Notifications (Optional)  
- Connect your Slack credential in the Slack node  
- Choose a channel or user to send error alerts to

