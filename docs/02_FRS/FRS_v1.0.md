# FRS v1.0 — Digital Onboarding (Individuals + SMEs) — KYC/KYB + CRM (Lead-first)

## 1. Document Control
- **Document Title:** Functional Requirements Specification (FRS)
- **Project:** Digital Onboarding Portal (Individuals + SMEs) with KYC/KYB + CRM Integration
- **Version:** v1.0 (Baseline)
- **Region Assumption:** EU (GDPR + AMLD context)
- **Owner:** Requirements Analyst (Portfolio Case Study)
- **Status:** Baseline
- **Last Updated:** 2026-02-12

> This FRS v1.0 baselines the functional behavior, integration expectations, workflows, and validations to support use cases, test cases, and traceability.

---

## 2. Purpose / Scope / Roles / Workflow / Requirements
This version baselines the content from FRS v0.1 and is intended to be the stable reference for:
- Use Case pack
- Test Case matrix
- Traceability Matrix (BR → FR → UC → TC)

**Baseline Policies**
- Region: EU (GDPR + AMLD context)
- Provider callback: Webhooks (HMAC signed + idempotent)
- Provider decision mapping: PASS → approve if eligible / REVIEW → manual review / FAIL → auto-reject
- SME screening scope: representative + all UBOs
- Incomplete UBO ownership disables auto-approval (routes to manual review)

---

## 3. Baselined Functional Requirements
The requirements set **FR-001 to FR-045** is baselined as defined in:
- `docs/02_FRS/FRS_v0.1.md` (sections 8–11)

No functional deltas introduced in v1.0 beyond baselining.

---

## 4. Linked Specifications
- BRD v1.0: `docs/01_BRD/BRD_v1.0.md`
- Data Dictionary: `docs/02_FRS/data-dictionary.md`
- Integration Specs: `docs/02_FRS/integration-specs.md`