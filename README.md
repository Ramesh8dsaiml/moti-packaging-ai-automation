# 📦 Moti Packaging Industries — Automated Lead Intake & Intelligence Pipeline

An end-to-end automated B2B sales lead qualification and extraction pipeline built using **n8n**, **OpenAI (GPT-4o-mini)**, **Python**, and **Google Sheets**.

The system ingests raw, unstructured B2B packaging enquiries across chat and email channels, intercepts duplicate leads, extracts 11 packaging specifications without hallucination, flags edge cases for human review, and populates a real-time executive analytics dashboard.

---

## 🚀 Business Problem & Operational Impact

**Moti Packaging Industries (MPI)** specializes in rigid composite paper canisters, paper tubes, and core containers. Previously, the sales intake workflow faced major bottlenecks:
* **High Latency:** 24–48 hours spent manually clarifying missing packaging dimensions (diameter, height, volume).
* **Wasted Capacity:** Sales engineers spending hours vetting out-of-scope enquiries (e.g., pure tinplate cans, plastic extrusion containers).
* **Data Discrepancies:** Multiple sales reps responding to duplicate enquiries.

### Measurable Results:
* **Lead Qualification Time:** Reduced from **48 hours to < 5 seconds**.
* **Sales Efficiency:** Filtered out 100% of out-of-scope and duplicate leads automatically.
* **Zero Hallucination:** Dimensions are never guessed; incomplete leads trigger automated follow-up drafts.

---

## 🛠️ Architecture & Pipeline Data Flow


```mermaid
flowchart TD
    A["📩 Customer Enquiry (Chat / Email)"] --> B{"🔍 Duplicate Detection (Python Cache)"}
    
    B -- "Duplicate" --> C["📋 Format Duplicate Leads"]
    B -- "New Lead" --> D["🤖 AI Extraction & Classification (GPT-4o-mini)"]
    
    D --> E["⚙️ Attach Enquiry ID & Format Leads (Python)"]
    E --> F[("📊 Google Sheets Database (Sheet1)")]
    C --> F
    
    F --> G["🎯 Sales Priority Funnel"]
    F --> H["📦 Product Category Breakdown"]
    F --> I["⚠️ Missing Specs Bottleneck"]

    style A fill:#EFF6FF,stroke:#3B82F6,stroke-width:2px,color:#1E3A8A
    style B fill:#FEF3C7,stroke:#F59E0B,stroke-width:2px,color:#92400E
    style C fill:#F1F5F9,stroke:#64748B,stroke-width:1.5px,color:#334155
    style D fill:#EDE9FE,stroke:#8B5CF6,stroke-width:2px,color:#5B21B6
    style E fill:#E0E7FF,stroke:#6366F1,stroke-width:1.5px,color:#3730A3
    style F fill:#DCFCE7,stroke:#10B981,stroke-width:2px,color:#065F46
    style G fill:#F8FAFC,stroke:#94A3B8,stroke-width:1px,color:#0F172A
    style H fill:#F8FAFC,stroke:#94A3B8,stroke-width:1px,color:#0F172A
    style I fill:#F8FAFC,stroke:#94A3B8,stroke-width:1px,color:#0F172A


### Core Workflow Components:
1. **Chat/Email Ingestion Trigger:** Captures raw, unstructured customer queries in freeform text.
2. **Duplicate Detection (Python):** Intercepts repeated customer enquiries against a seen-cache before sending to the LLM, eliminating redundant API token costs.
3. **AI Extraction & Classification (GPT-4o-mini):** 
   * Extracts 11 structured fields: Customer Name, Company, Contact (`email | phone`), Product Category, Diameter, Height, Quantity, Application, Printing, Delivery Location, and Required Date.
   * Enforces strict business classification: `Ready for quotation`, `More information required`, or `Not currently serviceable`.
   * Automatically generates tailored 2–3 sentence response drafts.
4. **Data Normalization (Python):** Cleans missing values into explicit `N/A`, `None`, and `YES/NO` flags for reliable database ingestion.
5. **Google Sheets Sync & Error Handling:** Appends normalized records to `Sheet1`, with a parallel `Failure Alert Logger` node catching and logging any execution drops.

---

## 📊 Live Analytics Dashboard

The database feeds into an automated executive dashboard featuring:
* **Enquiry Status Metrics:** Real-time tracking of Total Enquiries, Ready for Quotation, Information Pending, Not Serviceable, Duplicates, and Human Review flags.
* **Status Distribution Donut Chart:** Visual breakdown of pipeline health and qualification percentages.
* **Product Demand Column Chart:** Comparative volume analysis across Composite Canisters, TIN Cans, and Paper Tubes.
* **Missing Specs Bottleneck Analysis:** Granular tracking of missing technical dimensions (Diameter vs. Height vs. Quantity) to identify lead-funnel drop-offs.

---

## 📋 Key Assumptions & Guardrails

1. **Manufacturing Scope:** MPI exclusively manufactures composite paper cans, rigid paper tubes, and paper core containers. All pure metal, plastic, or corrugated requests are classified as `Not currently serviceable`.
2. **Zero-Hallucination Policy:** If diameter or height is omitted in the raw enquiry, the value is returned as `null` and flagged under `missing_fields`. The AI is never permitted to guess tooling sizes.
3. **Human-in-the-Loop Trigger:** High-value volumes, custom barrier requests, or non-serviceable items automatically set `human_review_required = YES`.

---

## ⚙️ How to Deploy & Test

### 1. n8n Workflow Import
1. Open your n8n workspace.
2. Go to **Workflows -> Import from File** and select `workflow.json`.
3. Configure your OpenAI credentials and Google Sheets OAuth connection.
4. Open the workflow chat window and test with sample enquiries.

### 2. Google Sheets Dashboard Generation
1. Open your target Google Sheet (`MPI Enquiries Dashboard`).