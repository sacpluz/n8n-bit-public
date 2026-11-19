# Modul 03.1 – Looping (Bonus)

Dieses Handout ist der Text der Sticky Notes aus dem Workflow, damit ihr ihn im zweiten Browser-Tab lesen und Code kopieren könnt.

**Workflow-Datei:** [`modul-03-1-looping-bonus.json`](modul-03-1-looping-bonus.json)

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

## Explanation of the n8n Nodes in the Workflow

### Manual Trigger  
This node starts the workflow manually and sends test data.  
**Documentation:** [Manual Trigger Node](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.manualtrigger/)

### Split Out (Split Array of Strings into Array of Objects)  
Extracts the `urls` array and splits it so that each URL becomes a separate JSON object.  
**Documentation:** [Split Out Node](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.splitout/)

### Loop Over Items (Loop Over Items 1)  
Demonstrates how an **unsplit** array is treated as one item, processing it as a single unit.  
**Documentation:** [Loop Over Items Node](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.splitinbatches/)

### Loop Over Items (Loop Over Items 2)  
Processes the **split array** one item at a time. This demonstrates individual iteration and controlled looping.  
**Documentation:** [Loop Over Items Node](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.splitinbatches/)

### Wait Node  
Introduces a **1-second delay** per iteration to demonstrate sequential execution. This can be removed for faster performance.  
**Documentation:** [Wait Node](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.wait/)

### Code Nodes (Add param1 to outputX)  
Each Code node adds a constant field (`param1`) to the data, ensuring that every item gets enriched with the same parameter.  
**Documentation:** [Code Node](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.code/)

### NoOp Nodes (Result1, Result2, Result3, Result4, Result5)
These nodes display the output at different stages of the workflow for inspection.
**Documentation:** [NoOp Node](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.noop/)



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

