# Use Cases v1.0 — Digital Onboarding (Individuals + SMEs) — KYC/KYB + CRM (Lead-first)

## 1. Document Control
- **Document Title:** Use Case Specification
- **Project:** Digital Onboarding Portal (Individuals + SMEs) with KYC/KYB + CRM Integration
- **Version:** v1.0
- **Region Assumption:** EU (GDPR + AMLD context)
- **Status:** Baseline
- **Last Updated:** 2026-02-12

## 2. Notes / Conventions
- Provider decisions:
  - **PASS** → auto-approve if eligible; otherwise manual review
  - **REVIEW** → manual review
  - **FAIL** → **auto-reject**
- SME screening scope: **Representative + all UBOs**
- UBO ownership incomplete (total ≠ 100% or unknown ownership) → **not eligible for auto-approval**

---

## 3. Actors
- **Applicant (Individual)**
- **Applicant (Business Representative)**
- **Operations Agent**
- **Compliance Analyst**
- **System Admin**
- **Onboarding System (Backend)**
- **Verification Provider (External System)**
- **CRM (External System)**

---

## 4. Use Case List
- **UC-001** Register and Verify Email (OTP)
- **UC-002** Create/Update Draft Application
- **UC-003** Submit Individual Application
- **UC-004** Submit SME Application (Company + Representative + UBOs)
- **UC-005** Upload/Replace Documents
- **UC-006** Initiate Provider Checks (KYC/KYB/Screening)
- **UC-007** Process Provider Decision Webhook (PASS/REVIEW/FAIL)
- **UC-008** Compliance Manual Review (Approve/Reject)
- **UC-009** Request More Info Loop (Compliance ↔ Applicant)
- **UC-010** CRM Lead Sync + Lead Conversion on Approval

---

# UC-001 Register and Verify Email (OTP)
**Primary Actor:** Applicant  
**Supporting Systems:** Notification Gateway (email)  
**Related FRs:** FR-001, FR-002, FR-003, FR-043

## Preconditions
- Applicant is not logged in.

## Trigger
- Applicant selects “Register” and enters email.

## Main Flow
1. Applicant enters email.
2. System sends OTP to the email.
3. Applicant enters OTP.
4. System validates OTP.
5. System marks email as verified and creates an authenticated session.

## Alternate Flows
- **A1: Resend OTP**
  - Applicant requests OTP resend.
  - System sends OTP (rate-limited per policy).

## Exception Flows
- **E1: OTP invalid**
  - If OTP incorrect, system shows error and increments failed attempts.
- **E2: OTP lockout**
  - After 3 consecutive failures, system locks OTP verification for 15 minutes.

## Postconditions
- Applicant account exists with `email_verified=true`.

---

# UC-002 Create/Update Draft Application
**Primary Actor:** Applicant  
**Related FRs:** FR-004, FR-005, FR-006, FR-007, FR-043

## Preconditions
- Applicant email is verified (UC-001 completed).

## Trigger
- Applicant starts onboarding.

## Main Flow
1. Applicant selects onboarding type: INDIVIDUAL or SME.
2. System displays relevant form sections based on type.
3. Applicant enters data (partial allowed).
4. Applicant selects “Save Draft”.
5. System generates/persists `application_id` (if first save), stores draft, status=Draft.

## Alternate Flows
- **A1: Resume Draft**
  - Applicant returns later and resumes using verified email/session.

## Exception Flows
- **E1: Draft expired**
  - If draft is older than 30 days, system marks it Expired and requests a new application.

## Postconditions
- Draft application exists and can be resumed.

---

# UC-003 Submit Individual Application
**Primary Actor:** Applicant (Individual)  
**Supporting Systems:** Provider, CRM  
**Related FRs:** FR-008..FR-011, FR-016..FR-020, FR-021, FR-023..FR-025, FR-036..FR-037, FR-041, FR-043

## Preconditions
- Application exists in Draft.
- Applicant has completed required fields and uploads required documents.

## Trigger
- Applicant clicks “Submit”.

## Main Flow
1. System validates required individual fields (name, DOB, address, consents).
2. System validates documents (ID + POA), file rules, AV scan status.
3. If validation passes, system sets status=Submitted and records `submitted_at`.
4. System creates provider checks (KYC + any required screening) (UC-006).
5. System sets status=Verification In Progress.
6. System creates/updates CRM Lead (Lead-first) with status and application_id.
7. System notifies applicant: “Application submitted.”

## Alternate Flows
- **A1: Duplicate detected**
  - If duplicate flagged, system still submits but sets `duplicate_flag=true` and blocks auto-approval eligibility.

## Exception Flows
- **E1: Mandatory field missing**
  - System blocks submission and shows field-level errors.
- **E2: Document upload invalid**
  - System blocks submission and indicates required document issues (type/size/AV fail/POA recency).

## Postconditions
- Application is in Verification In Progress or blocked for correction.

---

# UC-004 Submit SME Application (Company + Representative + UBOs)
**Primary Actor:** Applicant (Business Representative)  
**Supporting Systems:** Provider, CRM  
**Related FRs:** FR-008, FR-009, FR-011..FR-015, FR-016..FR-020, FR-021..FR-025, FR-036..FR-037, FR-041, FR-043

## Preconditions
- SME Draft application exists.
- Representative email verified.

## Trigger
- Representative clicks “Submit”.

## Main Flow
1. System validates required company fields (legal name, reg no., country, registered address).
2. System validates representative KYC fields (name, DOB >=18, address, consents).
3. System validates UBO entries (1..N minimum) and calculates ownership total.
4. System validates required documents:
   - Rep ID + Rep POA + Company Registration (and other policy docs).
5. System sets status=Submitted and records `submitted_at`.
6. System initiates provider checks: KYC(rep), KYB(company), Screening(rep + all UBOs) (UC-006).
7. System sets status=Verification In Progress.
8. System creates/updates CRM Lead with application_id and status.
9. System notifies applicant: “Application submitted.”

## Alternate Flows
- **A1: UBO ownership incomplete**
  - If ownership total ≠ 100% OR any unknown ownership flag exists:
    - System allows submission
    - Sets `not_eligible_auto_approval=true` (FR-014)

- **A2: Duplicate detected (company reg no.)**
  - System flags duplicate and blocks auto-approval eligibility.

## Exception Flows
- **E1: Missing required UBO**
  - Submission blocked if no UBO records present (policy: 1..N required).
- **E2: Document validation failure**
  - Submission blocked with document error details.

## Postconditions
- Application is in Verification In Progress or blocked for correction.

---

# UC-005 Upload/Replace Documents
**Primary Actor:** Applicant  
**Supporting Systems:** Document Storage, AV scanner  
**Related FRs:** FR-016..FR-020, FR-043

## Preconditions
- Application exists in Draft OR Awaiting Additional Info.

## Trigger
- Applicant uploads a document.

## Main Flow
1. Applicant selects document type and uploads file.
2. System validates file type and size.
3. System sends file to AV scan; if PASS, stores encrypted in document storage.
4. System persists document metadata and version.
5. System updates UI to show upload success and document status.

## Alternate Flows
- **A1: Replace document**
  - Applicant uploads a newer version of same document type.
  - System increments version and links to replaced_document_id.

## Exception Flows
- **E1: Unsupported file type/size**
  - System rejects upload with a clear error.
- **E2: AV scan FAIL**
  - System rejects file; logs audit event; prompts re-upload.

## Postconditions
- Document metadata exists; latest version is available for verification/review.

---

# UC-006 Initiate Provider Checks (KYC/KYB/Screening)
**Primary Actor:** Onboarding System (Backend)  
**Supporting Systems:** Verification Provider  
**Related FRs:** FR-021..FR-025, FR-023..FR-024, FR-043

## Preconditions
- Application is Submitted and required data/documents are present.

## Trigger
- Submission event.

## Main Flow
1. System builds provider payloads for required checks (based on INDIVIDUAL vs SME).
2. System calls provider create-check APIs using Idempotency-Key.
3. System stores provider reference IDs and request timestamps.
4. System sets status=Verification In Progress.

## Exception Flows
- **E1: Provider timeout/5xx**
  - System retries up to 3 times with backoff.
  - On final failure, create Ops alert and persist failure state.

## Postconditions
- Provider checks exist (or failure has been recorded and alerted).

---

# UC-007 Process Provider Decision Webhook (PASS/REVIEW/FAIL)
**Primary Actor:** Verification Provider (system)  
**Supporting Systems:** Onboarding System, CRM  
**Related FRs:** FR-026..FR-030, FR-027..FR-029, FR-037, FR-040..FR-043

## Preconditions
- Provider checks were created for the application.

## Trigger
- Provider sends decision webhook event.

## Main Flow
1. System receives webhook payload.
2. System validates webhook signature and timestamp.
3. System checks idempotency using event ID; ignores duplicates.
4. System stores raw payload for audit and updates the relevant check.
5. When all required checks are complete, system derives final application outcome:
   - If any required check = FAIL → status=Rejected (auto-reject)
   - Else if any required check = REVIEW → status=Under Manual Review
   - Else all PASS:
     - If auto-approval eligible → status=Approved
     - Else → status=Under Manual Review
6. System updates CRM Lead with new status and decision data.
7. System notifies applicant based on decision:
   - Approved / Rejected / Under Review

## Alternate Flows
- **A1: PASS but not eligible for auto-approval**
  - System routes to Under Manual Review even though provider returned PASS.

## Exception Flows
- **E1: Invalid signature**
  - System rejects webhook (401/403), logs security audit event, alerts engineering/ops.
- **E2: Payload cannot be matched**
  - If application_id/provider_verification_id unknown, log event and respond 422.

## Postconditions
- Application status is updated; audit history is preserved; CRM synced (or queued).

---

# UC-008 Compliance Manual Review (Approve/Reject)
**Primary Actor:** Compliance Analyst  
**Supporting Systems:** CRM, Notification Gateway  
**Related FRs:** FR-031..FR-035, FR-037..FR-040, FR-041..FR-043

## Preconditions
- Application status = Under Manual Review.

## Trigger
- Compliance Analyst opens manual review queue and selects a case.

## Main Flow
1. Analyst reviews applicant/company data, documents, and provider results.
2. Analyst selects decision: Approve or Reject.
3. Analyst selects decision reason code and adds notes (required for decision).
4. System records decision in DecisionHistory and AuditEvent.
5. System sets application status accordingly (Approved/Rejected).
6. System updates CRM Lead and triggers Lead conversion on approval (UC-010).
7. System notifies applicant of final decision.

## Exception Flows
- **E1: Unauthorized access**
  - If user is not in Compliance role, deny access and log an audit event.

## Postconditions
- Final decision recorded; CRM updated; applicant notified.

---

# UC-009 Request More Info Loop (Compliance ↔ Applicant)
**Primary Actor:** Compliance Analyst  
**Supporting Actor:** Applicant  
**Related FRs:** FR-033, FR-034, FR-041, FR-043

## Preconditions
- Application is Under Manual Review.

## Trigger
- Analyst selects “Request more info”.

## Main Flow
1. Analyst selects required items (fields and/or document types).
2. System sets status=Awaiting Additional Info.
3. System notifies applicant with request details.
4. Applicant uploads requested items (UC-005).
5. Applicant confirms “Resubmit”.
6. System records audit events and returns case to Under Manual Review queue.

## Alternate Flows
- **A1: Partial completion**
  - Applicant uploads some but not all items; system keeps status Awaiting Additional Info until complete.

## Exception Flows
- **E1: Applicant does not respond**
  - After configured period, system may expire/cancel the application (policy).

## Postconditions
- Requested items captured; case returns to compliance review.

---

# UC-010 CRM Lead Sync + Lead Conversion on Approval
**Primary Actor:** Onboarding System (Backend)  
**Supporting Systems:** CRM  
**Related FRs:** FR-036..FR-040, FR-043

## Preconditions
- Application has a CRM Lead ID (or needs creation).
- Application status changes OR final decision is reached.

## Trigger
- Status update event OR decision event.

## Main Flow
1. On Submit, system creates CRM Lead (or updates if already exists).
2. On status changes, system updates CRM Lead fields (onboarding_status, timestamps, provider refs).
3. On Approved, system triggers CRM Lead conversion (Lead → Account + Contact).
4. System stores returned CRM Account/Contact IDs in CRMMapping.
5. System logs CRM sync audit events.

## Exception Flows
- **E1: CRM 5xx/timeout**
  - System queues retry; marks crm_sync_status=PENDING_RETRY; alerts after threshold.
- **E2: CRM 4xx validation error**
  - System routes to data issue queue; marks FAILED_DATA_ISSUE; requires Ops/Admin fix.

## Postconditions
- CRM reflects onboarding status and outcomes; approved Leads are converted.

---