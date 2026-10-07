# Runtime_Terrors_HACKTOBER_FEST

# LedgerMind AI
### Explainable AI Accounting Copilot for Intelligent Voucher Classification

> **An AI-powered accounting assistant that understands financial transactions, predicts voucher categories, explains every decision, verifies its own entries, and assists accountants with intelligent recommendations using Open-Source LLMs.**

---

## Problem Statement

Accounting teams process thousands of structured financial transactions every day. Although transaction data is already available, identifying the correct voucher category still requires manual effort and accounting expertise.

Traditional rule-based systems fail when:

- Transactions are incomplete
- Similar voucher types exist
- Multiple fields must be analyzed together
- Business context is required

This results in slower workflows, inconsistent bookkeeping, and increased human effort.

---

# Our Solution

LedgerMind AI is an **Explainable AI Accounting Copilot** that combines accounting rules, machine learning, and Open-Source Large Language Models to understand the complete context of every transaction.

Instead of acting like a black-box classifier, LedgerMind AI:

- Predicts voucher types
- Writes and verifies the accounting entry behind each prediction
- Learns from accountant corrections
- Flags uncertain records for human review
- Assists accountants with actionable recommendations

---

# Key Features

✅ Intelligent Voucher Classification

✅ Explainable AI with confidence score

✅ Hybrid AI (Rules + ML + Open-Source LLM)

✅ Transaction Verification Engine

✅ Human-in-the-Loop Review

✅ Accountant Copilot Workspace

⭐ **Innovative Feature 1: Correction Memory**

Every time an accountant corrects a voucher label, LedgerMind AI saves that correction in a simple storage table. The table holds three columns: the transaction details, the correct voucher type, and a short note explaining the correction. When a new transaction arrives, the system searches this table for similar past transactions, such as the same supplier name or similar item descriptions. If a match is found, it is added to the AI's prompt as a worked example, for instance: *"Last time, this supplier's bill was a Purchase Return. Consider that here."* The AI then makes its decision with this context, so repeated mistakes are avoided without retraining the model.

⭐ **Innovative Feature 2: Double-Entry Self-Check**

Before a voucher type is accepted, the AI must also write the accounting entry it implies: which accounts are debited and credited, and by how much. A rule-based checker (plain code, no AI) then tests that entry. Debits must equal credits, the accounts must fit the voucher type (for example, a Contra entry touches only cash or bank accounts), and GST must flow in the right direction (input GST on purchases, output GST on sales). If the check fails, the AI receives the specific error and tries once more. If it still fails, the record is marked Uncertain with a low confidence score and sent for human review. The confidence score is therefore earned from the check result rather than guessed by the model.

---

# System Architecture

```text
Excel Dataset
      │
      ▼
Data Validation & Preprocessing
      │
      ▼
Correction Memory Lookup  ◄── Accountant Corrections Store
      │
      ▼
LLM Classifier (Open-Source LLM / SLM)
(Voucher Type + Dr/Cr Entry + Reasoning)
      │
      ▼
Double-Entry Self-Check (rule-based)
      │
      ├── Passed ──► Confidence & Risk Scoring
      │
      └── Failed ──► Retry once with error message ──► Still failing? ──► Mark Uncertain
      │
      ▼
Output (JSON / Excel)
      │
      ▼
Accountant Review ──► Corrections saved to Memory Store
```
---

# AI Workflow

```
Upload Dataset
      ↓
Validate Data
      ↓
Generate Features
      ↓
Voucher Classification
      ↓
Explain Prediction
      ↓
Detect Risks
      ↓
Recommend Next Action
      ↓
Export Results
```

---

# Technology Stack

| Category | Technology |
|----------|------------|
| **Frontend** | React.js, Tailwind CSS, AG Grid, Recharts |
| **Backend** | FastAPI, Python, Pandas, NumPy, OpenPyXL |
| **AI** | Gemma / Qwen / Llama / Mistral, LangChain |
| **ML** | Scikit-Learn, XGBoost / LightGBM |
| **Vector Search** | FAISS |
| **Database** | PostgreSQL |
| **Model Serving** | Ollama / vLLM |
| **Deployment** | Docker, GitHub Actions |
| **Version Control** | Git & GitHub |

---

# Why Open-Source AI?

- Enterprise-friendly
- Data privacy
- Local deployment
- Fine-tunable models
- Lower inference cost
- No vendor lock-in

---

# Expected Output

```json
{
  "transaction_id": "TX1001",
  "voucher_type": "Purchase",
  "confidence": 96,
  "reason": "Supplier sold inventory goods to the company.",
  "risk": "Low",
  "recommended_action": "Ready for Posting"
}
```

---

# Expected Impact

- Reduce manual accounting effort
- Improve voucher classification accuracy
- Detect financial inconsistencies early
- Increase transparency through explainable AI
- Enable scalable accounting automation

---

# Future Scope

- ERP Integration (Tally, SAP, Odoo, Zoho Books)
- AI Audit Assistant
- Continuous Learning from Accountant Feedback
- Multi-language Support
- Enterprise Cloud Deployment

---

# Team Vision

LedgerMind AI is not just a voucher classifier—it is an **AI Accounting Copilot** designed to assist finance professionals by combining intelligent reasoning, explainable predictions, and practical accounting insights.

Our goal is **not to replace accountants, but to make them faster, more accurate, and more confident in every financial decision.**
