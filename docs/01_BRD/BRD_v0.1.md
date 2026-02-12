# BRD v0.1 — Digital Onboarding (Individuals + SMEs) — KYC/KYB + CRM (Lead-first)

## 1. Document Control
- **Document Title:** Business Requirements Document (BRD)
- **Project:** Digital Onboarding Portal (Individuals + SMEs) with KYC/KYB + CRM Integration
- **Version:** v0.1 (Draft)
- **Region Assumption:** EU (GDPR + AMLD context)
- **Owner:** Requirements Analyst (Portfolio Case Study)
- **Status:** Draft
- **Last Updated:** 2026-02-12

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
- **Unified portal** onboarding for:
  - Individuals (personal onboarding)
  - SMEs (business onboarding via a representative)
- Applicant registration and **email OTP verification**
- Data capture:
  - Individual: personal details, address, contact, consents
  - SME: company details + representative details + UBO capture (1..N)
- Document upload and secure storage:
  - Individual: ID + Proof of Address
  - SME: representative ID + representative POA + company registration doc (+ company address proof as policy)
- External verification via **KYC/KYB Provider**:
  - KYC for individual / representative
  - KYB for company
  - **Sanctions/PEP screening for representative + all UBOs**
  - Decision callback via **webhooks**
- Manual review workflow:
  - Under Manual Review queue
  - Request additional info / documents
  - Approve / Reject with reason codes
- CRM integration (Lead-first):
  - Create/update CRM **Lead** for submitted applications
  - Convert Lead → Account + Contact on approval
- Notifications (email/SMS) for key events
- Audit logging + basic operational reporting

### 4.2 Out of Scope (for this portfolio project)
- Payments, funding, card issuing, account activation
- Marketing automation journeys
- Advanced analytics dashboards beyond basic ops reporting
- Contract generation / e-signature (future enhancement)

---

## 5. Stakeholders and Roles (Business View)
- **Business Owner / Head of Onboarding:** business goals, final scope, prioritization
- **Compliance Officer (AML/KYC):** verification policy, reason codes, evidence requirements
- **Operations Team Lead:** queue handling, exception management, SLAs
- **Customer Support Lead:** applicant communications and status inquiries
- **CRM Administrator:** CRM objects/fields, validation rules, Lead conversion
- **Backend/Integration Lead:** API contracts, webhook security, retries, monitoring
- **QA Lead:** test strategy and coverage
- **Security/Privacy (DPO):** GDPR requirements, retention, access controls

---

## 6. High-Level Business Process (To-Be)
1. Applicant registers and verifies email via OTP.
2. Applicant creates application (Individual or SME) and uploads required documents.
3. Applicant submits application.
4. System triggers provider checks (KYC/KYB + screenings) and waits for webhook decision.
5. System auto-approves eligible PASS cases; routes REVIEW/exception cases to Compliance.
6. FAIL decisions are **auto-rejected**.
7. CRM Lead is created/updated throughout; approved cases are converted to Account + Contact.
8. Applicant receives notifications at key milestones.

---

## 7. Business Requirements (BR)
- **BR-001** Support onboarding for **Individuals and SMEs** via a single digital portal.
- **BR-002** Perform **automated KYC** for individuals/representatives and **KYB** for SME companies.
- **BR-003** Screen representative and all UBOs for **Sanctions/PEP** and retain outcomes.
- **BR-004** Provide a **manual review** process: request-more-info, approve, reject, reason codes.
- **BR-005** Integrate onboarding with CRM using a **Lead-first** model to reduce manual entry.
- **BR-006** Provide applicant **status tracking** and notifications to reduce abandonment and support load.
- **BR-007** Securely collect, store, and manage required onboarding documents.
- **BR-008** Maintain an **audit trail** of actions, decisions, and integration events.
- **BR-009** Comply with **GDPR**: consent capture, access control, retention, traceability.
- **BR-010** Provide basic operational visibility: queue counts, aging, and failure alerts.
- **BR-011** Detect potential duplicates (person/company) and route for resolution.
- **BR-012** Ensure resilience for integrations so decisions are not lost (retries/queues).

---

## 8. Assumptions
- Provider supports creation of KYC/KYB checks and webhook callbacks for decisions.
- CRM supports APIs for Lead creation/update and Lead conversion.
- Documents can be stored encrypted; antivirus scanning is available (or simulated).
- Evidence retention is required for AML (assume **5 years**, configurable).

---

## 9. Dependencies
- KYC/KYB provider sandbox credentials + webhook signing secret
- CRM sandbox credentials and field configuration
- Email/SMS gateway configuration
- Document storage solution and AV scanning capability (or mocked service)

---

## 10. Constraints
- GDPR data minimization and controlled access to PII and documents
- Security: TLS in transit, encryption at rest, secrets management
- Auditability: events must be traceable to actor + timestamp
- Webhook events must be securely validated and processed idempotently

---

## 11. Success Criteria (Business Acceptance)
- 100% of submitted applications are represented in CRM as Leads.
- PASS decisions that meet eligibility rules are auto-approved with minimal manual effort.
- Manual review provides complete decision history with reason codes and audit trail.
- Applicants receive timely status notifications and can see next required actions.

---

## 12. High-Level Risks (summary)
- Webhook security/replay attacks
- Provider downtime causing backlog
- CRM mapping/validation issues causing sync failures
- GDPR exposure from improper access controls
- Incomplete UBO info increasing manual review volume

---

## 13. Sign-Off (Portfolio)
- Business Owner: __________________ Date: __________
- Compliance Owner: ________________ Date: __________
- Technical Owner: _________________ Date: __________