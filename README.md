# Automated Student Data Cleaning & ETL Workflow with n8n & Ollama LLM

An automated n8n workflow designed to perform Extract, Transform, Load (ETL) operations on student enrollment datasets. The workflow receives input files (`.csv` or `.xlsx`) via an n8n Form trigger, uses a local LLM (**Ollama** with `qwen2.5:0.5B-F16`) to clean and standardize student records, normalizes the data with custom JavaScript logic, and segregates output files based on student location criteria.

---

## 📌 Overview

This workflow automates student dataset ingestion and data hygiene pipelines:

1. **Extraction (Form Trigger & File Extractors)**:
   - Receives uploaded files through an n8n Form interface (`FileUpload`).
   - Routes files dynamically using a **Switch** node based on extension (`.csv` vs. `.xlsx`).
   - Extracts tabular data into individual JSON records.

2. **Transformation (LLM & Data Sanitization)**:
   - Batches incoming records (5 items per batch) via a **Loop Over Items** node.
   - Leverages **Ollama** (`qwen2.5:0.5B-F16`) in a **Basic LLM Chain** to enforce cleaning rules (capitalization, date formatting, course name normalization, email validation, phone number verification, and fee status booleans).
   - Runs a post-processing **JavaScript Code Node** to parse raw model responses, decode special characters/entities, sanitize placeholder values (converting `"UNKNOWN"`, `"N/A"`, `"NULL"` to `null`), and enforce strict Title Case formatting.

3. **Loading & Segregation (Conditional Routing)**:
   - Filters processed records using an **If** node based on email validity and geographical location (`City == Chennai`).
   - Converts clean records into two distinct binary export files:
     - **`students_in_chennai`**: Valid records for students located in Chennai.
     - **`out_of_chennai_students`**: Records for students outside Chennai or with invalid email addresses.

---

## 🛠️ Workflow Architecture

```
[ Form Trigger (FileUpload) ]
             │
             ▼
      [ Switch Node ]
     ├── .csv  ➜ [ Extract from File (CSV)  ] ──┐
     └── .xlsx ➜ [ Extract from File1 (XLSX) ] ──┤
                                                │
                                                ▼
                                    [ Loop Over Items (Batch: 5) ]
                                                │
                                                ▼
                                   [ Basic LLM Chain ] ◄── [ Ollama Chat Model ]
                                                │            (qwen2.5:0.5B-F16)
                                                ▼
                                    [ Code in JavaScript ]
                                                │
                                                ▼
                                             [ If ]
                                    ┌──────────┴──────────┐
                            (Chennai & Valid)       (Non-Chennai / Invalid)
                                    │                     │
                                    ▼                     ▼
                          [ Convert to File ]   [ Convert to File1 ]
                          (students_in_chennai) (out_of_chennai_students)
```

---

## 📋 Transformation & Data Cleaning Rules

The LLM prompt and JavaScript post-processor enforce strict transformation logic:

| Field | Rule / Transformation | Example Input | Example Output |
| :--- | :--- | :--- | :--- |
| **Name** | Title case every word. | `"mANOJ kUMAR"` | `"Manoj Kumar"` |
| **Email** | Trim whitespace, lowercase, validate standard syntax. Invalid emails return `"INVALID_EMAIL"`. | `"MANOJ.K@GMAIL.COM"` | `"manoj.k@gmail.com"` |
| **Phone** | Must contain at least 10 digits. Returns `"MISSING"` if fewer. | `"98765"` | `"MISSING"` |
| **Course** | Normalize case-insensitively to `"Python"`, `"Machine learning"`, or `"Data Science"`. | `"ml"` | `"Machine learning"` |
| **Fee_Paid** | Converts variations of `"yes"`/`"no"` to native JSON booleans `true`/`false`. | `"YES"` | `true` |
| **City** | Title case string. If null/empty, returns `"UNKNOWN"`. | `"chennai"` | `"Chennai"` |
| **Enrolled_Date** | Standardizes valid, unambiguous dates into ISO `YYYY-MM-DD` format. | `"23-05-2025"` | `"2025-05-23"` |

---

## ⚙️ Prerequisites

1. **n8n Instance**: n8n v1.0 or higher.
2. **Ollama Instance**:
   - Local or remote Ollama server accessible to n8n (default credential endpoint, e.g., `http://localhost:11434`).
   - Model pulled and available: `docker.io/ai/qwen2.5:0.5B-F16` (or compatible small language model).

---

## 🚀 Setup & Installation

### 1. Import Workflow

1. Open your **n8n Dashboard**.
2. Navigate to **Workflows** > **Add Workflow**.
3. Click the **`⋮` (Menu)** in the upper-right corner and choose **Import from File**.
4. Select `Basic ETL automation.json`.

### 2. Credentials Configuration

- **Ollama API**: Double-click **Ollama Chat Model** node and select or create your Ollama API credential ID.

### 3. Usage

1. Activate the workflow or open the **On form submission** node.
2. Open the generated Form URL in your browser.
3. Upload a student enrollment file (`.csv` or `.xlsx`).
4. Execute the workflow to review filtered binary files containing sanitized data outputs."# n8n_ETL_Basics" 
