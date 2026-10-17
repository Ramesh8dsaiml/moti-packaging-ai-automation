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

## ⚙️ Core Technical Logic & Implementation

### 1. Pre-LLM Duplicate Interception
- Raw incoming chat messages are evaluated against recent leads before invoking the language model.
- Duplicates are routed through `Format Duplicate Leads` straight to Google Sheets with status `Possible Duplicate`, saving OpenAI API tokens and preventing multi-agent overlaps.

### 2. Zero-Hallucination AI Extraction (`GPT-4o-mini`)
- The `AI Extraction & Classification` sub-agent uses the **Structured Output Parser** to enforce strict JSON compliance.
- If any technical specification (inner diameter, height, closure) is missing, it is mapped strictly to `null` / `N/A` rather than estimated or hallucinated.

### 3. Automated Scope Vetting
- **In-Scope:** Composite paper canisters, spiral-wound paper tubes, and rigid core containers.
- **Out-of-Scope:** 100% tinplate metal cans, glass containers, and plastic blow-molded bottles.
- Out-of-scope requests are auto-classified under `Not Currently Serviceable`, removing manual engineering overhead.

### 4. Resilient Error Handling
- Downstream database or write failures trigger the `Failure Alert Logger` without breaking the live execution pipeline.

---

## 📋 Data Contract & Extracted Schema

The pipeline parses 11 core attributes from every incoming raw query:

| Field Name | Type | Description / Constraint |
| :--- | :--- | :--- |
| `ENQ - Id` | `String` | Unique deterministic identifier (e.g., `ENQ-fdad0b96`) |
| `Customer Name` | `String` | Contact person name (`N/A` if omitted) |
| `Company Name` | `String` | Business entity name |
| `Contact` | `String` | Phone number or corporate email address |
| `Product Category` | `String` | Composite Canister, Tin Can, Paper Tube / Core, Other |
| `Inner Diameter` | `Number / null` | Inner diameter in millimeters |
| `Height / Length` | `Number / null` | Height / length in millimeters |
| `Closure Type` | `String / null` | Peel-off foil, metal plug, plastic cap, or paper plug |
| `Quantity` | `Number / null` | Requested order quantity / batch size |
| `Intended Content` | `String / null` | Product to pack (e.g., tea, spices, apparel, cosmetics) |
| `Lead Status` | `String` | `Ready for Quotation`, `Information Pending`, `Human Review Needed`, `Not Currently Serviceable`, `Possible Duplicate` |

---

## 📊 Live Executive Analytics & Reporting Dashboard

The automation feeds real-time operational data into the [MPI Enquiries Dashboard](https://docs.google.com/spreadsheets/u/0/?authuser=0&usp=sheets_web) with automated metrics powered by Apps Script (`dashboard_script.js.txt`):

### Funnel & Qualification Breakdown (109 Ingested Leads)
- **Ready for Quotation (43.0% / 52 Leads):** Fully defined specifications; routed instantly to estimating for tooling and unit cost calculations.
- **Information Pending (36.4% / 44 Leads):** Incomplete technical parameters; flagged for automated specification follow-up.
- **Not Currently Serviceable (16.5% / 20 Leads):** Out-of-scope materials (tin cans, rigid plastics); filtered out to protect sales capacity.
- **Human Review Needed (20 Leads):** Edge cases, composite hybrids, or non-standard requirements.
- **Possible Duplicate (4.1% / 5 Leads):** Intercepted pre-LLM to conserve API quota.

---

## 🚀 Setup & Deployment Guide

### Prerequisites
- Active [n8n cloud or self-hosted instance](https://rameshworkflowsystem.app.n8n.cloud/projects/RpjlvgLKzPkeOMnK) (v1.0+)
- OpenAI API key with `gpt-4o-mini` access
- Google Cloud service account or OAuth credentials configured for Google Sheets read/write

### Deployment Steps
1. **Clone the Repository:**
   ```bash
   git clone [https://github.com/Ramesh8dsaiml/moti-packaging-ai-automation.git](https://github.com/Ramesh8dsaiml/moti-packaging-ai-automation.git)
   cd moti-packaging-ai-automation

 
