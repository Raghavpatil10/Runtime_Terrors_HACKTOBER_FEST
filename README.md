# LedgerMind AI

### Explainable AI Accounting Copilot for Intelligent Voucher Classification

> An AI-powered accounting assistant that classifies transactions into voucher types, writes and verifies the accounting entry behind each prediction, learns from accountant corrections, and flags uncertain records for human review, using open-source LLMs.

Team: Runtime_Terrors | Event: Hacktoberfest 4 (Elevate), Qualifier Round

---

## 2. Problem Statement

Accounting teams process thousands of structured financial transactions every day. Although the transaction data already exists, deciding the correct voucher category (Purchase, Sales, Payment, Contra, Journal, and so on) still requires manual effort and accounting expertise.

Traditional rule-based systems fail when:

- Transactions are incomplete
- Similar voucher types look alike (Purchase vs Sales, Purchase Return vs Sales Return)
- Several fields must be read together to decide the type
- Business context, such as a supplier's usual pattern, is needed

The result is slower workflows, inconsistent bookkeeping, and more human effort.

---

## 3. Project Overview

LedgerMind AI takes an Excel file of structured transactions and returns, for each row, a voucher type, the debit and credit entry behind it, a confidence score, and a short reason. Every prediction passes through a rule-based check before it is accepted. Records that cannot be verified are marked Uncertain and sent to an accountant. Each accountant correction is stored and used as an example for future transactions.

---

## 4. Proposed Solution

LedgerMind AI combines three parts:

- **Open-source LLM** (Qwen2.5-7B-Instruct, Apache 2.0 license) reads the full transaction context and proposes a voucher type with its accounting entry.
- **Rule-based verification** (plain Python, no AI) checks that the entry is valid: debits equal credits, accounts fit the voucher type, and GST flows in the correct direction.
- **Correction memory** stores accountant corrections and feeds similar past cases back into the prompt.

Together these make the system explainable, self-checking, and improving over time, without retraining the model.

---

## 5. Objectives

1. Classify each transaction into one of the 28 target voucher categories using an open-source model.
2. Produce a balanced, rule-checked accounting entry for every accepted prediction.
3. Reduce repeated errors by reusing past accountant corrections.
4. Route uncertain or invalid records to human review instead of accepting them silently.
5. Output structured JSON and Excel results that can be evaluated programmatically.

---

## 6. Target Users / Use Case

**Primary users:** accounting teams and bookkeepers at small and medium businesses who receive transaction exports (Excel or CSV) from billing or ERP systems and need them sorted into voucher types before entry into accounting software.

**Use case:** An accountant uploads a month's transaction file. LedgerMind AI classifies each row, marks high-confidence records as ready for posting, and lists flagged records for review. The accountant's corrections are saved, so the same supplier or pattern is handled better next time.

---

## 7. Open-Source AI Technology Selected

- **Model:** Qwen2.5-7B-Instruct (Apache 2.0). A quantized version runs locally.
- **Serving:** Ollama for local inference during development; vLLM for a larger deployment.
- **Orchestration:** LangChain for prompt construction and structured output parsing.
- **Embeddings for memory search:** a small open-source sentence-embedding model, indexed with FAISS.

---

## 8. Why This Technology Was Selected

- **Qwen2.5-7B-Instruct** follows instructions and produces structured JSON well at a size that runs on a single consumer GPU or a quantized CPU setup. Its Apache 2.0 license allows commercial and enterprise use without restrictions.
- **Local serving** (Ollama / vLLM) keeps financial data on the company's own machine, which matters for accounting data.
- **FAISS** is a lightweight, open-source vector index that works without a separate server.
- **Rule-based verification in plain Python** is used for the checks because accounting rules are exact. An LLM is not needed to decide whether debits equal credits, so it is not asked to.

---

## 9. AI's Role in the System

The AI does three things and nothing else:

1. **Classifies** the transaction into a voucher type, using the full context (supplier, buyer, amount, GST, description, and similar past corrections).
2. **Writes** the debit and credit entry that the chosen voucher type implies.
3. **Explains** the choice in one or two sentences.

The AI does not decide whether its own output is correct. The rule-based checker does that, and the AI only gets one retry based on the checker's specific error message.

---

## 10. System Architecture

```text
Excel / CSV Upload
        │
        ▼
Data Validation & Preprocessing
        │
        ▼
Correction Memory Lookup  ◄── Accountant Corrections Store
        │
        ▼
LLM Classifier (Qwen2.5-7B-Instruct)
(Voucher Type + Dr/Cr Entry + Reasoning)
        │
        ▼
Double-Entry Self-Check (rule-based)
        │
        ├── Passed ──► Confidence & Risk Scoring
        │
        └── Failed ──► Retry once with error message
                            │
                            ├── Passed ──► Confidence & Risk Scoring
                            └── Failed ──► Mark Uncertain (low score)
        │
        ▼
Output (JSON / Excel)
        │
        ▼
Accountant Review ──► Corrections saved to Memory Store
```

---

## 11. Component-Level Architecture

| Component | Responsibility | Technology |
|-----------|----------------|------------|
| Input Handler | Reads Excel/CSV, validates columns, handles missing values and duplicates | Python, Pandas, OpenPyXL |
| Preprocessor | Normalizes supplier names, dates, amounts, and GST fields; builds a transaction context string | Python, Pandas |
| Correction Memory | Stores corrections as (transaction details, correct voucher type, note); retrieves similar cases | PostgreSQL, FAISS |
| LLM Classifier | Produces voucher type, Dr/Cr entry, and reasoning | Qwen2.5-7B-Instruct via Ollama/vLLM, LangChain |
| Double-Entry Checker | Verifies debits = credits, account-to-voucher fit, and GST direction | Plain Python rules |
| Risk & Confidence Scorer | Combines check result, memory agreement, and simple anomaly rules into a score and risk level | Python |
| API Layer | Exposes upload, results, and correction endpoints | FastAPI |
| Review Workspace | Lets an accountant view results, accept or correct them, and see flagged items first | React.js, Tailwind CSS, AG Grid |

---

## 12. Data / Information Flow

1. An accountant uploads an Excel or CSV file.
2. The preprocessor cleans the rows and builds a context for each transaction.
3. For each row, the correction memory returns up to three similar past corrections.
4. The LLM receives the transaction context and those examples, and returns a voucher type, a Dr/Cr entry, and reasoning in JSON.
5. The checker tests the entry. If it fails, the error message goes back to the LLM for one retry.
6. The confidence and risk scorer produces the final score and recommended action.
7. Results are exported as JSON and Excel, and shown in the review workspace.
8. Accountant corrections are written to the memory store with the transaction details and a short note.

---

## 13. Agentic Workflow (if applicable)

Not applicable. The workflow is a fixed pipeline with one bounded retry. The model does not choose its own tools or plan multi-step actions, so no agent loop is used. This keeps behavior predictable and easy to audit.

---

## 14. Technology Stack

| Category | Technology |
|----------|------------|
| Frontend | React.js, Tailwind CSS, AG Grid, Recharts |
| Backend | FastAPI, Python, Pandas, NumPy, OpenPyXL |
| AI Model | Qwen2.5-7B-Instruct |
| AI Orchestration | LangChain |
| Model Serving | Ollama (development), vLLM (deployment) |
| Rule Checker | Python (plain code, no AI) |
| Memory Search | FAISS, sentence-embedding model |
| Database | PostgreSQL |
| Deployment | Docker, GitHub Actions |
| Version Control | Git & GitHub |

---

## 15. Expected Features

- **Voucher classification** into 28 target categories, from one Excel or CSV upload. Each row is processed on its own and returns one voucher type.
- **Confidence score and risk level** for every record. The score combines the self-check result, agreement with correction memory, and simple anomaly rules such as duplicates or unusual amounts.
- **Uncertain routing.** Records that fail the self-check twice are marked Uncertain with a low score and placed in the review queue instead of being accepted.
- **Duplicate and missing-field detection** during preprocessing. Duplicate rows are flagged, and missing required fields such as supplier, amount, or GST are reported before classification.
- **Human-in-the-loop review workspace.** Flagged and Uncertain records appear first, and the accountant can accept the suggested voucher or correct it.
- **JSON and Excel export** of all results, including the voucher type, debit and credit entry, confidence, reason, and recommended action.

⭐ **Innovative Feature 1: Correction Memory**

Every time an accountant corrects a voucher label, LedgerMind AI saves that correction in a simple storage table. The table holds three columns: the transaction details, the correct voucher type, and a short note explaining the correction. When a new transaction arrives, the system searches this table for similar past transactions, such as the same supplier name or similar item descriptions. If a match is found, it is added to the AI's prompt as a worked example, for instance: *"Last time, this supplier's bill was a Purchase Return. Consider that here."* The AI then makes its decision with this context, so repeated mistakes are avoided without retraining the model.

⭐ **Innovative Feature 2: Double-Entry Self-Check**

Before a voucher type is accepted, the AI must also write the accounting entry it implies: which accounts are debited and credited, and by how much. A rule-based checker (plain code, no AI) then tests that entry. Debits must equal credits, the accounts must fit the voucher type (for example, a Contra entry touches only cash or bank accounts), and GST must flow in the right direction (input GST on purchases, output GST on sales). If the check fails, the AI receives the specific error and tries once more. If it still fails, the record is marked Uncertain with a low confidence score and sent for human review. The confidence score is therefore earned from the check result rather than guessed by the model.

---

## 16. Implementation Approach

**Phase 1: Data and rules.** Build the Excel reader and preprocessor. Write the rule checker first, with the account-to-voucher mapping for the most common types (Purchase, Sales, Payment, Receipt, Contra, Journal, and their returns).

**Phase 2: Classifier.** Set up Qwen2.5-7B-Instruct locally with Ollama. Write the prompt template that requests JSON with voucher type, entry, and reasoning. Test on a small labelled sample.

**Phase 3: Memory and retry.** Add the corrections table and FAISS search. Add the single retry loop with the checker's error message.

**Phase 4: Review and evaluation.** Build the review workspace, the correction save function, and an evaluation script that reports accuracy, precision, recall, F1, and per-category results on unseen records.

**Phase 5: Packaging.** Dockerize the services and document how to reproduce the evaluation.

---

## 17. Expected Final Output

For each transaction, the system produces:

```json
{
  "transaction_id": "TX1001",
  "voucher_type": "Purchase",
  "entry": {
    "debit": "Purchases 50000, Input GST 9000",
    "credit": "Sharma Traders 59000"
  },
  "check_status": "Passed",
  "confidence": 96,
  "reason": "Supplier sold inventory goods to the company.",
  "risk": "Low",
  "memory_match": "TX0874 (Sharma Traders, corrected to Purchase Return)",
  "recommended_action": "Ready for Posting"
}
```

Records that fail the check twice return `"check_status": "Failed"`, `"voucher_type": "Uncertain"`, and a low confidence score, with `"recommended_action": "Human review"`.

---

## 18. Future Scope / Scalability

- ERP integration (Tally, SAP, Odoo, Zoho Books)
- Continuous learning from accountant feedback, with periodic fine-tuning using LoRA on saved corrections
- AI audit assistant for flagged records
- Multi-language support for invoice descriptions
- Scalability: the classifier can run on multiple GPU workers, and the memory index can grow to millions of records. The rule checker is fast and runs on every record without extra cost.

---

## 19. Open-Source Dependencies / Components

| Component | Purpose | License |
|-----------|---------|---------|
| Qwen2.5-7B-Instruct | Classification model | Apache 2.0 |
| Ollama | Local model serving | MIT |
| vLLM | Scalable model serving | Apache 2.0 |
| LangChain | LLM orchestration | MIT |
| FAISS | Vector similarity search | MIT |
| Sentence-Transformers | Text embeddings for memory search | Apache 2.0 |
| FastAPI | Backend API | MIT |
| Pandas, NumPy | Data processing | BSD-3-Clause |
| OpenPyXL | Excel reading and writing | MIT |
| PostgreSQL | Correction memory storage | PostgreSQL License |
| React.js | Frontend | MIT |
| Tailwind CSS | Styling | MIT |
| AG Grid (Community) | Results table | MIT |

---

## 20. Expected Challenges and Mitigation

| Challenge | Mitigation |
|-----------|------------|
| Semantically similar voucher types (Purchase vs Sales, Contra vs Payment) | The double-entry checker rejects entries that do not match the type; correction memory supplies examples for known confusions |
| Incomplete or ambiguous records | Missing fields are flagged in preprocessing; low-confidence records go to human review instead of being guessed |
| Model output not valid JSON | Structured output parsing with schema validation; one retry, then Uncertain |
| Wrong entries that look plausible | The rule checker tests balance and account fit independently of the model's opinion |
| Confidence scores poorly calibrated | Scores are based on check results and memory agreement, not the model's raw probability |
| Limited hardware for a 7B model | Use a quantized build; process in batches; fall back to a smaller model for development |
| Sensitive financial data | Local deployment only; no data is sent to external APIs |
| Small or noisy correction memory early on | Memory is used only when similarity passes a threshold; the system works without it |

## Team Details
1. Raghav Patil (Leader)
2. Aryan Jumde
3. Shamit bundela
4. Priyanshu Yadav

