# Digital Onboarding (Individuals + SMEs) — KYC/KYB + CRM (Lead-first) Requirements Pack

Fictional case study created to demonstrate end-to-end **Requirements Engineering** deliverables for a system that supports:
- Customer onboarding for **Individuals** and **SMEs**
- **KYC (person)** and **KYB (company)** checks via an external provider
- **Sanctions/PEP screening** for the representative and all UBOs
- **Webhook-based** verification decision processing (secure + idempotent)
- CRM integration using a **Lead-first** model (convert Lead → Account + Contact on approval)
- Manual review workflow for Compliance exceptions

> Disclaimer: This repository is for learning/portfolio purposes only. All names, processes, and data are fictional.

---

## What’s included (artifacts)
- **BRD** (Business Requirements Document): objectives, scope, stakeholders, assumptions, constraints  
- **FRS** (Functional Requirements Specification): functional requirements, validations, workflows, integrations, NFRs  
- **Use Cases**: main/alternate/exception flows  
- **Test Cases**: input → expected output validation matrix  
- **RTM (Traceability Matrix)**: BR → FR → UC → TC mapping  
- **Risk Register**: risks + mitigation/contingency  

---

## Repository structure
- `docs/01_BRD/`
  - `BRD_v0.1.md` (draft)
  - `BRD_v1.0.md` (baseline)
- `docs/02_FRS/`
  - `FRS_v0.1.md` (draft)
  - `FRS_v1.0.md` (baseline)
  - `data-dictionary.md`
  - `integration-specs.md`
- `docs/03_UseCases/`
  - `UseCases_v1.0.md`
- `docs/04_Testing/`
  - `TestCases.csv`
  - `TraceabilityMatrix.csv`
- `docs/05_Risks/`
  - `RiskRegister_v1.0.md`
- `docs/06_Diagrams/`
  - workflow diagrams (draw.io/PNG)
- `change-log.md`

---

## Key design choices (scenario policies)
- Region assumed: **EU** (GDPR + AMLD context)
- Verification callback: **Webhooks**
- Provider decision handling:
  - **PASS** → auto-approve if eligible; otherwise manual review
  - **REVIEW** → manual review
  - **FAIL** → **auto-reject**
- SME screening scope: **Representative + all UBOs**
- UBO ownership completeness:
  - submission allowed even if ownership total ≠ 100% or unknown
  - such cases are **not eligible for auto-approval** and route to manual review

---

## How to review quickly (recommended reading order)
1. `docs/01_BRD/BRD_v1.0.md`
2. `docs/02_FRS/FRS_v1.0.md`
3. `docs/03_UseCases/UseCases_v1.0.md`
4. `docs/04_Testing/TestCases.csv`
5. `docs/04_Testing/TraceabilityMatrix.csv`
6. `docs/05_Risks/RiskRegister_v1.0.md`

---

## Author
- Sanskar Sinha
