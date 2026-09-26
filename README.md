# Pre-MLR Compliance Copilot: AI Content Generation & Compliance Pre-Check (n8n + LLM)

An automated compliance pre-screening pipeline that generates on-brand marketing copy and immediately verifies it against a defined set of regulatory rules — combining deterministic rule matching with dual-pass LLM review — before it ever reaches a human Medical/Legal/Regulatory (MLR) reviewer.

![Pre MLR Compliance Copilot n8n + LLM](<Pre MLR Compliance Copilot n8n + LLM.jpg>)

## Overview

Marketing teams in regulated industries (life sciences, finance, and similar) must run every piece of promotional content through a formal compliance review before publishing — checking for unsupported claims, missing legal disclosures, invented statistics, non-compliant incentives, and unbalanced benefit statements. This review is typically slow, manual, and repeated for every asset produced.

This **Pre-MLR Compliance Copilot** removes the bottleneck of the *first* pass. Using a webhook-triggered n8n workflow, LLM-based content generation, a deterministic rule engine, and a dual-pass AI reviewer, generated marketing copy is automatically checked against every compliance rule and returned as a structured, evidence-based report — so human reviewers spend their time on genuine edge cases instead of routine checks.

---

## Architecture & Workflow

```text
                        [ Webhook: Campaign Brief + Source Material ]
                                          │
                                          ▼
                        ┌─────────────────────────────────┐
                        │      CONTENT GENERATION (LLM)    │
                        │   • Email + landing page copy    │
                        │   • Constrained to source facts  │
                        └────────────────┬──────────────────┘
                                          │
                    ┌─────────────────────┴─────────────────────┐
                    ▼                                             ▼
        ┌────────────────────────┐                  ┌─────────────────────────────┐
        │   DETERMINISTIC RULES   │                  │     AI COMPLIANCE REVIEW     │
        │  • Required disclosures │                  │   (dual independent passes)  │
        │  • Required CTA wording │                  │  • Topic accuracy            │
        │  • Word-count limits    │                  │  • Unsupported claims        │
        └────────────┬─────────────┘                  │  • Invented statistics       │
                     │                                 │  • Incentive/kickback risk   │
                     │                                 │  • Fair-balance requirements │
                     │                                 └──────────────┬───────────────┘
                     │                                                │
                     └───────────────────────┬────────────────────────┘
                                              ▼
                              ┌───────────────────────────────┐
                              │     CONSENSUS + MERGE LAYER    │
                              │  • Rule-by-rule evidence       │
                              │  • Worst-case-wins AI consensus│
                              └───────────────┬─────────────────┘
                                              ▼
                              ┌───────────────────────────────┐
                              │   STRUCTURED COMPLIANCE REPORT │
                              │  • PDF / Word export           │
                              │  • Human reviewer sign-off      │
                              └─────────────────────────────────┘
```

### 1. Content Generation

- **Webhook Ingestion**: Accepts a campaign brief (objective, creative direction, optional custom instruction) plus an approved source-of-truth document.
- **LLM Copywriter**: Generates coordinated email + landing page copy, constrained to only use facts present in the supplied source material — unsupported requests are declined and reframed rather than fabricated.

### 2. Dual-Layer Compliance Review

- **Deterministic Rule Engine**: Exact-match checks (required disclosure text, required call-to-action wording, word-count limits) — fully repeatable, zero variance.
- **AI Compliance Reviewer**: Semantic checks (topic accuracy against source material, unsupported/fabricated claims, invented statistics or endorsements, non-compliant incentives, fair-balance requirements) — run as **two independent passes**, merged with a worst-case-wins consensus rule to reduce single-pass inconsistency inherent to LLM-based review.

### 3. Structured Reporting

- Every rule returns a status, plain-language explanation, cited source evidence, and a suggested correction.
- Reports are rendered as a shareable HTML page and exportable to PDF/Word for human reviewer sign-off.

---

## Structured Output Schema

Each AI-judged rule returns a strict, schema-enforced object to guarantee deterministic downstream merging:

```json
{
  "rule_id": "R7",
  "status": "issues_flagged",
  "asset": "email",
  "wording": "Register today and receive a complimentary $50 gift card.",
  "explanation": "This constitutes an inducement for attendance, which is not allowed.",
  "source_reference": "CR7 — Do not offer gifts, cash, prizes, meals, or other items of value as an inducement.",
  "suggested_correction": "Remove the inducement offer from the email.",
  "human_review_action": "Ensure compliance with anti-kickback regulations by removing the inducement."
}
```

---

## Repository Structure

```text
├── workflow/
│   └── pre_mlr_compliance_copilot.json     # Full n8n workflow export
├── docs/
│   └── testing_report.pdf                  # Full regression + edge-case test log
├── Pre MLR Compliance Copilot n8n + LLM.jpg
└── README.md
```

---

## Key Features

- **Dual-Layer Verification**: Deterministic rule matching for zero-variance checks, paired with AI-based semantic review for judgment-based compliance rules.
- **Self-Consistency Safety Net**: Runs AI-based review twice independently per submission and merges on worst-case-wins logic, converting single-pass LLM inconsistency into a materially higher effective catch rate.
- **Evidence-Based Reporting**: Every flagged issue is returned with cited source evidence and a suggested correction — no black-box scoring.
- **Live Content Updates**: Supports mid-review content changes (e.g. event date/time updates) with automatic re-verification.
- **Full Regression Test Suite**: Baseline behavior, isolated single-rule checks, combined-violation cross-contamination tests, update-flow edge cases, and stability checks — every fix verified across multiple repeated runs before sign-off.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Workflow Orchestration | n8n (Cloud) |
| Content Generation | OpenAI API (LangChain node) |
| Compliance Review (AI) | OpenAI API — dual independent passes |
| Deterministic Rule Engine | JavaScript (custom Code nodes) |
| Data Storage | Supabase |
| Report Rendering | HTML → PDF/Word export |
