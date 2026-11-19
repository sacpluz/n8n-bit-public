# Modul 03.1 – Looping (Bonus) – Erklärungen

Was der Trainer am Beamer erklärt. Die Aufgaben stehen in der [README.md](README.md).

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
