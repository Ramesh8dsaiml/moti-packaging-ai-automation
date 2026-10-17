 # 📦 Moti Packaging Industries — Automated Lead Intake & Intelligence Pipeline

An end-to-end automated B2B sales lead qualification and extraction pipeline built using **n8n**, **OpenAI (GPT-4o-mini)**, **Python / JavaScript Code Nodes**, and **Google Sheets**.

The system ingests raw, unstructured B2B packaging enquiries across chat and email channels, filters duplicate requests before costly LLM execution, extracts 11 technical packaging specifications with zero hallucination, flags edge cases for human review, and populates a real-time executive analytics dashboard.

---

## 🚀 Business Problem & Operational Impact

**Moti Packaging Industries (MPI)** specializes in rigid composite paper canisters, paper tubes, and core containers. Previously, sales intake faced critical bottlenecks:
* **High Latency:** 24–48 hours spent manually clarifying missing packaging dimensions (diameter, height, volume).
* **Wasted Capacity:** Sales engineers spending hours vetting out-of-scope enquiries (e.g., pure tinplate cans, plastic extrusion containers).
* **Data Discrepancies:** Multiple sales reps responding to duplicate enquiries across different shifts.

### Measurable Results:
* **Lead Qualification Time:** Reduced from **48 hours to < 5 seconds**.
* **API Cost Optimization:** Intercepts and formats duplicate leads early, bypassing redundant LLM calls.
* **100% Scope Vetting:** Automatically separates serviceable composite packaging leads from out-of-scope metal/plastic requests.
* **Zero Hallucination Guarantee:** Technical dimensions are strictly mapped to `null`/`N/A` if omitted; AI never guesses tooling sizes.

---

## 🛠️ Pipeline Architecture & Actual Node Flow

```mermaid
flowchart TD
    A[📩 When chat message received] --> B[🔍 Duplicate Detection]
    B --> C{Is Not Duplicate?}
    
    %% Duplicate Path
    C -- False (Duplicate) --> D[📋 Format Duplicate Leads]
    D --> E[(📊 Save to Google Sheets)]
    
    %% New Lead Path
    C -- True (New Lead) --> F[🤖 Classify and draft reply]
    subgraph F [Classify and draft reply Agent]
        F1[AI Extraction & Classification]
        F2[OpenAI Chat Model]
        F3[Structured Output Parser]
        F4[Split Out]
        F1 --- F2
        F1 --- F3
        F1 --> F4
    end
    
    F --> G[⚙️ Attach Enquiry ID]
    G --> H[🧹 Format Processed Leads]
    H --> E
    
    %% Error Catching
    E -. Error Catch .-> I[⚠️ Failure Alert Logger]


---

## 🧩 Detailed Node Breakdown

1. **Trigger (`When chat message received`):** Captures incoming customer messages and enquiry text from chat widgets, webhooks, or test payloads.
2. **Duplicate Detection:** Cross-checks incoming customer contact details and enquiry payload before triggering the LLM, preventing duplicate leads and conserving API tokens.
3. **Is Not Duplicate (Branching Node):**
   * **`False` branch:** Diverts directly to **`Format Duplicate Leads`**, marks status as `Duplicate Lead`, and bypasses LLM execution.
   * **`True` branch:** Passes the fresh enquiry into the AI classification agent.
4. **Classify and draft reply (AI Scope):**
   * **`AI Extraction & Classification` + `OpenAI Chat Model (GPT-4o-mini)`:** Extracts 11 structured parameters: *Customer Name, Company Name, Contact Info, Product Category, Diameter, Height, Quantity, Application, Printing Details, Delivery Location, and Required Delivery Date*.
   * **`Structured Output Parser`:** Validates JSON schema output and strictly assigns classification:
     * `Ready for quotation` (all critical manufacturing dimensions are present)
     * `More information required` (diameter, height, or critical spec is missing)
     * `Not currently serviceable` (pure metal tinplate, plastic, or out-of-scope products)
   * Drafts an automated 2–3 sentence reply tailored to the extraction result.
   * **`Split Out`:** Normalizes batch outputs into clean individual item records.
5. **Attach Enquiry ID & Format Processed Leads:** Appends sequential tracking IDs (e.g., `MPI-LEAD-XXXX`), sanitizes null values into explicit `N/A`, and sets standard `YES`/`NO` flags.
6. **Save to Google Sheets & Failure Alert Logger:** Appends structured rows directly into the Google Sheets database (`Sheet1`). If an API timeout, quota, or network failure occurs, the parallel `Failure Alert Logger` node catches and records execution errors.

---

## 📊 Executive Analytics Dashboard

The central Google Sheet is coupled with a custom Google Apps Script (`dashboard_script.js.txt`) that automatically builds an executive intelligence dashboard:
* **Live KPI Metric Cards:** Total Ingested Enquiries, Ready for Quotation, More Info Required, Out of Scope, Duplicates, and Human Review flags.
* **Status Distribution Donut Chart:** Breakdown of qualification health across all incoming traffic.
* **Product Demand Breakdown:** Comparative volume across Composite Canisters, Paper Tubes, and Tin Cans.
* **Missing Specs Bottleneck Funnel:** Granular metrics showing where leads drop off (e.g., missing diameter vs. height vs. order volume).

---

## 📋 Key Assumptions & Guardrails

1. **Manufacturing Scope:** MPI exclusively produces rigid composite paper canisters, paper tubes, and paper core containers. All non-paperboard/pure metal tinplate or plastic injection requests are strictly classified as `Not currently serviceable`.
2. **Zero-Hallucination Policy:** If diameter, height, or barrier coating is not explicitly stated in customer text, it is returned as `null` and flagged in `missing_fields`. The AI never guesses tooling sizes.
3. **Human-in-the-Loop Trigger:** High order quantities (> 50,000 units), custom barrier requirements, or non-serviceable leads automatically set `human_review_required = YES`.

---

## ⚙️ Deployment & Setup Instructions

### 1. Workflow Import in n8n
1. Download or clone this repository.
2. In your n8n workspace, navigate to **Workflows -> Import from File** and select `workflow.json.json`.
3. Set up credentials for:
   * **OpenAI API** (configured with `gpt-4o-mini` or preferred chat model)
   * **Google Sheets OAuth2** (with write permissions to your target spreadsheet)
4. Verify node connections and hit **Publish** or test in the interactive chat window.

### 2. Live Dashboard Setup (Google Sheets)
1. Open your target Google Sheet.
2. Go to **Extensions -> Apps Script**.
3. Copy the script code from `dashboard_script.js.txt`.
4. Paste it into the Apps Script editor and click **Run** on the `createMPIDashboard` function.
5. The `Executive Dashboard` tab will instantly generate formatted KPI cards, summary tables, and dynamic charts.
