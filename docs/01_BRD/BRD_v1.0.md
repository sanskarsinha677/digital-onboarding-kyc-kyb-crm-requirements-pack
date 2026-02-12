# BRD v1.0 — Digital Onboarding (Individuals + SMEs) — KYC/KYB + CRM (Lead-first)

## 1. Document Control
- **Document Title:** Business Requirements Document (BRD)
- **Project:** Digital Onboarding Portal (Individuals + SMEs) with KYC/KYB + CRM Integration
- **Version:** v1.0 (Baseline)
- **Region Assumption:** EU (GDPR + AMLD context)
- **Owner:** Requirements Analyst (Portfolio Case Study)
- **Status:** Baseline
- **Last Updated:** 2026-02-12

> This BRD v1.0 baselines the business scope and high-level requirements for downstream functional specification and testing artifacts.

---

## 2. Background / Problem Statement
EuroFin Services GmbH (fictional EU fintech) currently performs customer onboarding using manual steps across web forms, email exchanges, spreadsheets, separate KYC/KYB tools, and re-keying data into CRM. This creates:
- Long onboarding cycle times
- Data inconsistencies and duplicate records
- Limited applicant visibility into status
- Compliance/audit gaps (decisions and evidence not consistently traceable)

The business requires a digital onboarding solution that automates verification, integrates with CRM, supports manual compliance review, and maintains an audit-ready trail.

---

## 3. Business Objectives (KPIs)
1. **Time-to-onboard:** Auto-approve low-risk cases in **< 30 minutes** end-to-end.
2. **Reduce manual entry:** Reduce manual CRM data entry by **≥ 90%** through automated sync.
3. **Compliance readiness:** Store verification outcomes, evidence, and decisions with complete auditability (GDPR + AMLD).
4. **Customer experience:** Provide transparent status tracking and proactive notifications to reduce abandonment and support load.

---

## 4. Scope

### 4.1 In Scope
- **Unified portal** onboarding for Individuals and SMEs
- Email OTP verification
- Data capture: Individual + SME + UBO capture (1..N)
- Document upload and secure storage
- KYC/KYB Provider integration via webhooks
- Sanctions/PEP screening for representative + all UBOs
- Manual review workflow (Compliance)
- CRM integration (Lead-first) + Lead conversion on approval
- Notifications + audit logging + basic operational reporting

### 4.2 Out of Scope
- Payments/funding/cards/account activation
- Marketing automation
- Advanced analytics dashboards
- Contract generation/e-signature

---

## 5. Stakeholders and Roles
- Business Owner / Head of Onboarding
- Compliance Officer (AML/KYC)
- Operations Team Lead
- Customer Support Lead
- CRM Administrator
- Backend/Integration Lead
- QA Lead
- Security/Privacy (DPO)

---

## 6. High-Level Business Process (To-Be)
Register → Draft → Submit → Verify (provider) → Webhook decision:
- PASS → auto-approve if eligible else manual review
- REVIEW → manual review
- FAIL → auto-reject  
CRM Lead created/updated; conversion on approval.

---

## 7. Business Requirements (BR)
- **BR-001** Support onboarding for Individuals and SMEs via a single digital portal.
- **BR-002** Perform automated KYC (person) and KYB (company).
- **BR-003** Screen representative and all UBOs for Sanctions/PEP and retain outcomes.
- **BR-004** Provide manual review with request-more-info, approve, reject, reason codes.
- **BR-005** Integrate onboarding with CRM using Lead-first model and conversion on approval.
- **BR-006** Provide applicant status tracking and notifications.
- **BR-007** Secure document collection and storage.
- **BR-008** Maintain full audit trail of actions/decisions/integration events.
- **BR-009** GDPR compliance (consent, access control, retention, traceability).
- **BR-010** Operational visibility (queues, aging, failures).
- **BR-011** Duplicate detection and resolution workflow.
- **BR-012** Integration resilience to avoid losing decisions (retries/queues).

---

## 8. Assumptions, Dependencies, Constraints, Success Criteria
Same as BRD v0.1; baselined for downstream design/testing.

---

## 9. Sign-Off (Portfolio)
- Business Owner: __________________ Date: __________
- Compliance Owner: ________________ Date: __________
- Technical Owner: _________________ Date: __________