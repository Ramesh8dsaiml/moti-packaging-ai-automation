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

```

## ⚙️ Core Technical Capabilities & Logic

### 1. Pre-LLM Deduplication Layer
- Intercepts incoming messages before triggering the OpenAI model to eliminate redundant token consumption.
- Matches incoming buyer contact details and enquiry context against recent entries.
- Directly routes duplicates via `Format Duplicate Leads` into Google Sheets with a `Possible Duplicate` status flag.

### 2. High-Precision Extraction (`GPT-4o-mini`)
- Implements strict schema enforcement through the **Structured Output Parser**.
- **Anti-Hallucination Guardrail:** The model never assumes or invents packaging dimensions. If any parameter (inner diameter, height, closure style) is omitted by the buyer, it is strictly assigned `null` / `N/A`.

### 3. Automatic Scope Vetting
- **Serviceable (In-Scope):** Rigid composite paper canisters, spiral wound paper tubes, composite core containers.
- **Non-Serviceable (Out-of-Scope):** 100% metal tinplate cans, glass jars, blow-molded plastic containers.
- Non-serviceable leads are systematically categorized as `Not Currently Serviceable`, allowing the sales engineering team to bypass manual review.

### 4. Resilient Execution & Fallback
- Downstream database write errors or connection timeouts trigger the `Failure Alert Logger` node.
- Captures payload metadata and error stack traces without crashing the parent execution cycle.

---

## 📋 Data Contract & Extracted Fields

The extraction engine parses 11 structured parameters from every inbound query:

| Field Name | Type | Description / Constraints |
| :--- | :--- | :--- |
| `ENQ - Id` | `String` | Deterministic unique enquiry identifier (e.g., `ENQ-fdad0b96`) |
| `Customer Name` | `String` | Contact person name (`N/A` if omitted) |
| `Company Name` | `String` | Registered organization or buyer entity |
| `Contact` | `String` | Email address, phone number, or handle |
| `Product Category` | `String` | Composite Canister, Tin Can, Paper Tube / Core, or Other |
| `Inner Diameter` | `Number / null` | Inner diameter in millimeters |
| `Height / Length` | `Number / null` | Tube length / can height in millimeters |
| `Closure Type` | `String / null` | Peel-off foil membrane, tin lid, plastic cap, or paper plug |
| `Quantity` | `Number / null` | Requested batch volume / order quantity |
| `Intended Content` | `String / null` | Product to be packaged (e.g., tea, spices, apparel, cosmetics) |
| `Lead Status` | `String` | `Ready for Quotation`, `Information Pending`, `Human Review Needed`, `Not Currently Serviceable`, or `Possible Duplicate` |

---

## 📊 Live Executive Analytics & Reporting Dashboard

The automation populates a multi-tab Google Sheet ([MPI Enquiries Dashboard](https://docs.google.com/spreadsheets/d/1VHJo5Zhi2eK74F8xpFFRyPZAhCC_slG2dTTAKpB4zNc/edit?gid=1304431208#gid=1304431208)) integrated with Apps Script (`dashboard_script.js.txt`):

### Operational Pipeline Funnel
* **Total Inbound Enquiries Processed:** `109`
* **Ready for Quotation (43.0% / 52 Leads):** Fully defined dimensions; routed instantly to estimating for tooling and unit cost calculations.
* **Information Pending (36.4% / 44 Leads):** Incomplete technical parameters; flagged for automated specification follow-up.
* **Not Currently Serviceable (16.5% / 20 Leads):** Out-of-scope materials (tin cans, rigid plastics); filtered out to protect sales capacity.
* **Human Review Needed (20 Leads):** Edge cases, composite hybrids, or non-standard requirements.
* **Possible Duplicate (4.1% / 5 Leads):** Filtered out pre-LLM to conserve API quota.

---

## 🚀 Setup & Deployment Guide

### Prerequisites
- Active [n8n cloud or self-hosted instance](https://rameshworkflowsystem.app.n8n.cloud/workflow/hzhkhuAJ6iZrDacc) (v1.0+)
- OpenAI API key with access to `gpt-4o-mini`
- Google Cloud service account or OAuth credentials configured for Google Sheets read/write

### Deployment Steps
1. **Clone Repository:**
   ```bash
   git clone [https://github.com/Ramesh8dsaiml/moti-packaging-ai-automation.git](https://github.com/Ramesh8dsaiml/moti-packaging-ai-automation.git)
   cd moti-packaging-ai-automation
