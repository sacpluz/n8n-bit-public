# Modul 03.1 – Looping (Bonus)

Dieses Handout ist der Text der Sticky Notes aus dem Workflow, damit ihr ihn im zweiten Browser-Tab lesen und Code kopieren könnt.

**Workflow-Datei:** [`modul-03-1-looping-bonus.json`](modul-03-1-looping-bonus.json)

**Erklärungen zum Modul** (was der Trainer am Beamer zeigt, zum Nachlesen): [TRAINER.md](TRAINER.md)

---

# N8N for Beginners: Looping Over Items

## Description

This workflow is designed for **n8n beginners** to understand how n8n handles **looping (iteration)** over multiple items. It highlights two key behaviors:

- **Built-In Looping:** By default, most n8n nodes iterate over each item in an input array.
- **Explicit Looping:** The **Loop Over Items** node allows controlled iteration, enabling **custom batch processing** and multi-step workflows.

This workflow demonstrates the difference between processing an **unsplit array of strings (single item)** vs. **a split array (multiple items)**.

---

## Setup

### 1. Input Data

To begin, **paste the following JSON** into the **Manual Trigger** node:

```json
{
  "urls": [
    "https://www.reddit.com",
    "https://www.n8n.io/",
    "https://n8n.io/",
    "https://supabase.com/",
    "https://duckduckgo.com/"
  ]
}
```

📌 **Steps to Paste Data:**
- **Double-click** the "Manual Trigger" node.
- Click **"Edit Output"** (top-right corner).
- Paste the JSON and **Save**.
- The node **turns purple**, indicating that test data is pinned. 

---

## Execution Details

### 1. How the Workflow Runs

- **Manual Trigger starts execution** with the pasted JSON data.
- The workflow follows **two paths**:

#### Unsplit Array Path â†’ Loop Over Items 1  
- The original JSON is processed **as a single item**.  
- **Result1 & Result5:** Show that the array was **not split** before processing.

#### Split Array Path â†’ Split Out â†’ Loop Over Items 2  
- The **Split Out** node converts the `urls` array into separate objects.
- The **Loop Over Items 2** node processes each URL **one by one**.
- A **Wait node** (1-second delay) demonstrates **controlled execution**.
- **Code nodes** modify the JSON, adding the field (`param1`).
- **Result2, Result3, Result4:** Display the final processed output.

### 2. What You Will See

- **Result1 & Result5:** The entire array is processed **as one item** before splitting.
- **Result2, Result3, Result4:** Each URL is processed **individually** after being split.
- **Wait Node:** Adds a **1-second delay per item** in **Loop Over Items 2**.

---

## Notes

- Sticky notes are included in the workflow **for easy reference**.
- The **Wait node** is **optional**â€”remove it for faster execution.
- This template is structured for **beginners** but serves as a **building block** for more advanced automations.

---
