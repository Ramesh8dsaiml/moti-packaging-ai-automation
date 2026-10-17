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




 
