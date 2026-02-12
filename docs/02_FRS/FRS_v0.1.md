# FRS v0.1 — Digital Onboarding (Individuals + SMEs) — KYC/KYB + CRM (Lead-first)

## 1. Document Control
- **Document Title:** Functional Requirements Specification (FRS)
- **Project:** Digital Onboarding Portal (Individuals + SMEs) with KYC/KYB + CRM Integration
- **Version:** v0.1 (Draft)
- **Region Assumption:** EU (GDPR + AMLD context)
- **Owner:** Requirements Analyst (Portfolio Case Study)
- **Status:** Draft
- **Last Updated:** 2026-02-12

---

## 2. Purpose
Define detailed functional and technical requirements for a portal that supports:
- Individual onboarding (KYC + screening)
- SME onboarding (KYB + representative KYC + UBO capture + screening)
- Provider webhooks for decisions (secure + idempotent)
- Manual review workflow for Compliance
- CRM integration using a **Lead-first** model (convert Lead → Account + Contact on approval)
- Audit logging, notifications, and operational reporting

---

## 3. Scope
### 3.1 In Scope
- Registration (email OTP), application creation, save draft/resume
- Data capture: Individual + SME + UBOs
- Document upload, validation, AV scan, secure storage, versioning
- Provider integration for:
  - KYC (individual/rep)
  - KYB (company)
  - Sanctions/PEP screening (rep + all UBOs)
  - Decision webhooks (PASS/REVIEW/FAIL)
- Manual review + request-more-info loop
- CRM: create/update Lead; convert on approval
- Notifications and audit trail

### 3.2 Out of Scope
- Payments/funding/cards/account activation
- Marketing automation
- Advanced BI dashboards (beyond basic ops reporting)
- E-signature

---

## 4. References
- BRD v1.0: `docs/01_BRD/BRD_v1.0.md`
- Data Dictionary: `docs/02_FRS/data-dictionary.md`
- Integration Specs: `docs/02_FRS/integration-specs.md`

---

## 5. Definitions / Glossary
- **KYC:** Know Your Customer (person identity verification)
- **KYB:** Know Your Business (company verification)
- **UBO:** Ultimate Beneficial Owner
- **PEP:** Politically Exposed Person
- **PII:** Personally Identifiable Information
- **Idempotency:** Processing the same event/request multiple times without duplicate effects

---

## 6. Actors / Roles (RBAC)
- **Applicant (Individual)**
- **Applicant (Business Representative)**
- **Operations Agent**
- **Compliance Analyst**
- **System Admin**
- **System Integration Accounts** (provider webhook caller, CRM API user)

---

## 7. High-Level Workflow & Status Model

### 7.1 Application Statuses
- **Draft**
- **Submitted**
- **Verification In Progress**
- **Under Manual Review**
- **Awaiting Additional Info**
- **Approved**
- **Rejected**
- **Cancelled / Expired**

### 7.2 Decision Policy (Provider)
- **PASS** → Auto-approve if eligible, else **Manual Review**
- **REVIEW** → **Manual Review**
- **FAIL** → **Auto-Reject**

### 7.3 Auto-Approval Eligibility Rules
Auto-approval is allowed only if all are true:
1. Provider decision = PASS for required checks (KYC/KYB + screenings)
2. No duplicate flags requiring Ops resolution
3. Documents valid (AV scan PASS, Proof-of-Address within policy)
4. UBO ownership completeness:
   - total ownership = 100%
   - no `ownership_unknown_flag = true`

If any condition fails, route to **Under Manual Review** (even if PASS).

---

## 8. Functional Requirements (FR)

> Format: **FR-xxx** — requirement statement + key rules / acceptance notes.

### 8.1 Registration & Identity (Applicant Access)
- **FR-001** The system shall allow applicants to register using email and verify via OTP.
- **FR-002** The system shall lock OTP verification for 15 minutes after 3 consecutive failed attempts.
- **FR-003** The system shall allow an applicant to resume onboarding using the verified email identity.

### 8.2 Application Management
- **FR-004** The system shall allow selection of onboarding type: **INDIVIDUAL** or **SME**.
- **FR-005** The system shall allow saving an application as **Draft** and resuming for up to 30 days.
- **FR-006** The system shall assign a unique `application_id` at first save and persist it through the lifecycle.
- **FR-007** The system shall display current status and next required actions to the applicant.

### 8.3 Data Capture & Validation (Inputs/Outputs/Rules)
- **FR-008** The system shall validate mandatory fields based on onboarding type prior to submission.
- **FR-009** The system shall enforce applicant/representative age **>= 18 years** at submission.
- **FR-010** The system shall validate email format and phone format (E.164 where provided).
- **FR-011** For SME onboarding, the system shall require company legal name, registration number, country, and registered address.
- **FR-012** For SME onboarding, the system shall capture **1..N UBO** records with required minimum fields (name, DOB, nationality) and ownership%.
- **FR-013** The system shall calculate total UBO ownership percentage and store the result.
- **FR-014** If UBO ownership total ≠ 100% OR any UBO has ownership_unknown_flag = true, the system shall mark the application as **not eligible for auto-approval**.
- **FR-015** The system shall perform duplicate detection:
  - Individuals: email/phone/ID (if captured)
  - SMEs: company registration number
  and flag matches for Ops review.

### 8.4 Documents (Upload, Validation, Storage)
- **FR-016** The system shall support document upload for allowed file types (PDF/JPG/PNG) and enforce a configurable max file size (default 10MB).
- **FR-017** The system shall perform antivirus scanning on uploaded documents and block processing if scan fails.
- **FR-018** The system shall enforce required documents by onboarding type:
  - Individual: ID + Proof of Address (POA)
  - SME: Representative ID + Representative POA + Company Registration Doc (+ Company Address Proof per policy)
- **FR-019** The system shall enforce POA recency: document date must be within the last 3 months (policy configurable).
- **FR-020** The system shall retain document versions for re-uploads and maintain an audit record of replacements.

### 8.5 Verification Orchestration (Provider)
- **FR-021** On submission, the system shall create verification requests with the provider:
  - KYC for Individual or Representative
  - KYB for SME company
- **FR-022** The system shall screen **Representative + all UBOs** for sanctions/PEP.
- **FR-023** The system shall store provider reference IDs per check and correlate them to `application_id`.
- **FR-024** The system shall set status to “Verification In Progress” after sending verification requests.
- **FR-025** The system shall implement retry logic for provider API failures (timeouts/5xx) with exponential backoff (3 attempts) and create an Ops alert on final failure.

### 8.6 Webhooks (Security, Idempotency, Processing)
- **FR-026** The system shall expose a webhook endpoint to receive verification decisions from the provider.
- **FR-027** The system shall validate webhook authenticity using HMAC signature verification (shared secret) and reject invalid signatures.
- **FR-028** The system shall process webhooks idempotently using provider event ID and/or (`provider_verification_id`, `application_id`) to prevent duplicate state changes.
- **FR-029** The system shall persist the raw webhook payload and processing result for audit purposes.
- **FR-030** The system shall update application status based on provider outcome:
  - PASS → Approved if eligible; otherwise Under Manual Review
  - REVIEW → Under Manual Review
  - FAIL → Rejected (auto-reject)

### 8.7 Manual Review (Compliance Case Handling)
- **FR-031** The system shall provide a Manual Review queue with filtering (status, aging, risk band, onboarding type).
- **FR-032** Compliance Analyst shall be able to approve/reject an application and must select a decision reason code and enter notes.
- **FR-033** Compliance Analyst shall be able to request additional information/documents and set status “Awaiting Additional Info”.
- **FR-034** The applicant shall be able to upload additional documents requested by Compliance and resubmit for review.
- **FR-035** The system shall maintain an immutable decision history (auto + manual) including timestamps and actor identity.

### 8.8 CRM Integration (Lead-first)
- **FR-036** On submit, the system shall create or update a CRM **Lead** with core applicant/company fields and onboarding status.
- **FR-037** The system shall update the CRM Lead on key status changes (Verification In Progress, Under Manual Review, Awaiting Additional Info, Approved, Rejected).
- **FR-038** On approval, the system shall trigger CRM **Lead conversion** to create **Account + Contact** and store the conversion result IDs back in the onboarding system.
- **FR-039** On rejection, the system shall not convert the Lead and shall update the Lead with rejection reason code(s).
- 