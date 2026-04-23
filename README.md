# Artikate_ML_Engineer-Task

# Written Systems Design Review

---

# Question B — Evaluating LMM Output Quality

deployed an internal report summarization model and need an evaluation framework beyond subjective feedback.

---

## 1. Build a Ground Truth Dataset

create a manually reviewed dataset of **500–1,000 internal reports** sampled across:

- short reports
- long reports
- technical reports
- operational reports
- finance reports
- noisy documents

For each report:

- human-written reference summary
- key facts checklist
- report metadata

Example:

```json
{
  "report": "...",
  "reference_summary": "...",
  "critical_facts": [
    "revenue declined 12%",
    "supply chain delay"
  ]
}
