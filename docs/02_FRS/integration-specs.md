# Integration Specifications — KYC/KYB Provider + CRM (Lead-first)

## 1. Purpose
This document defines the integration behavior for:
- **Verification Provider** (KYC/KYB + sanctions/PEP screening)
- **Webhook decision callbacks** (secure + idempotent)
- **CRM integration (Lead-first)**: create/update Lead and convert on approval

This is a fictional, portfolio-grade specification intended to be consistent with:
- `docs/02_FRS/FRS_v1.0.md`
- `docs/02_FRS/data-dictionary.md`

---

## 2. Integration Overview

### 2.1 Systems & Responsibilities
| System | Responsibility |
|---|---|
| Onboarding Portal/UI | Captures user input and uploads documents |
| Onboarding Backend | Orchestrates workflow, calls provider + CRM, receives webhooks, persists audit |
| Verification Provider | Executes KYC/KYB + screening and returns a decision |
| CRM (Salesforce-like) | Stores Leads and converts to Account/Contact on approval |
| Notification Gateway | Sends emails/SMS on key status events |
| Document Storage | Stores encrypted documents; provides references/tokens |

### 2.2 Integration Matrix
| Integration | Direction | Protocol | Auth |
|---|---|---|---|
| Onboarding → Provider (Create checks) | Outbound | HTTPS REST | API key / OAuth token (assumed) |
| Provider → Onboarding (Decision webhook) | Inbound | HTTPS REST | HMAC signature (shared secret) |
| Onboarding → CRM (Lead create/update/convert) | Outbound | HTTPS REST | OAuth2 / API token (assumed) |
| Onboarding → Notification | Outbound | HTTPS REST | API key (assumed) |

---

## 3. Common Requirements (all integrations)
### 3.1 Correlation & Observability
- All outbound requests MUST include:
  - `X-Correlation-Id: <application_id>`
- All logs/audit events SHOULD include:
  - `application_id`
  - external reference IDs (provider_verification_id, crm_lead_id)

### 3.2 Timeouts & Retries
- Default request timeout: **10 seconds**
- Retry policy for transient failures:
  - Retries on: HTTP **5xx**, network errors, timeouts
  - Do not retry on: HTTP **4xx** (except 429 rate limit)
  - Backoff: exponential (e.g., 1m, 5m, 15m) for up to **3 attempts**
- After final failure:
  - Create Ops alert
  - Persist failure details (sanitized)
  - Keep application decision safe (do not lose final state)

### 3.3 Idempotency
- Outbound create/update calls MUST support idempotency to avoid duplicates.
- Use `Idempotency-Key` header:
  - Format example: `APP:<application_id>:<operation>:<attempt_group>`
  - Example: `APP:a1b2:create_kyc:v1`

---

# 4. Verification Provider Integration (KYC/KYB + Screening)

## 4.1 Assumed Provider API Base
- Sandbox: `https://api.provider.example/v1`

> Note: Endpoint names are representative; adapt to any real provider contract.

## 4.2 Authentication (assumption)
- Outbound calls use:
  - `Authorization: Bearer <provider_access_token>`
  - or `X-Api-Key: <key>`

## 4.3 Create Verification Checks

### 4.3.1 Create KYC (Individual / Representative)
**Endpoint:** `POST /verifications/kyc`

**Headers**
- `Authorization: Bearer <token>`
- `Idempotency-Key: APP:<application_id>:KYC`
- `X-Correlation-Id: <application_id>`
- `Content-Type: application/json`

**Request Body (example)**
```json
{
  "application_id": "a1b2c3d4-....",
  "subject": {
    "role": "INDIVIDUAL",
    "first_name": "Anya",
    "last_name": "Meyer",
    "dob": "1990-01-01",
    "nationality": "DE",
    "address": {
      "line1": "Street 1",
      "city": "Berlin",
      "postal_code": "10115",
      "country": "DE"
    }
  },
  "documents": [
    { "type": "IDENTITY_DOCUMENT", "document_ref": "doc_123_token" },
    { "type": "PROOF_OF_ADDRESS", "document_ref": "doc_456_token", "document_date": "2026-01-15" }
  ],
  "callback": {
    "url": "https://onboarding.example/webhooks/provider/decision"
  }
}
Response (example)

JSON

{
  "provider_verification_id": "pv_kyc_001",
  "status": "PENDING",
  "created_at": "2026-02-12T10:00:00Z"
}
4.3.2 Create KYB (Company)
Endpoint: POST /verifications/kyb

Headers

Same as KYC, with Idempotency-Key: APP:<application_id>:KYB
Request Body (example)

JSON

{
  "application_id": "a1b2c3d4-....",
  "company": {
    "legal_name": "Meyer Consulting GmbH",
    "registration_number": "HRB123456",
    "country_of_incorporation": "DE",
    "registered_address": {
      "line1": "Business Str 10",
      "city": "Berlin",
      "postal_code": "10115",
      "country": "DE"
    },
    "vat_number": "DE123456789"
  },
  "documents": [
    { "type": "COMPANY_REGISTRATION_DOCUMENT", "document_ref": "doc_789_token" }
  ],
  "callback": {
    "url": "https://onboarding.example/webhooks/provider/decision"
  }
}
Response

JSON

{
  "provider_verification_id": "pv_kyb_001",
  "status": "PENDING",
  "created_at": "2026-02-12T10:01:00Z"
}
4.3.3 Create Screening (Sanctions/PEP) — Representative + All UBOs
Endpoint: POST /verifications/screening

Headers

Idempotency-Key: APP:<application_id>:SCREENING
Request Body (example)

JSON

{
  "application_id": "a1b2c3d4-....",
  "subjects": [
    {
      "subject_id": "rep_001",
      "role": "REPRESENTATIVE",
      "first_name": "Anya",
      "last_name": "Meyer",
      "dob": "1990-01-01",
      "nationality": "DE"
    },
    {
      "subject_id": "ubo_001",
      "role": "UBO",
      "first_name": "Jon",
      "last_name": "Meyer",
      "dob": "1980-02-02",
      "nationality": "DE"
    }
  ],
  "screening_types": ["SANCTIONS", "PEP"],
  "callback": {
    "url": "https://onboarding.example/webhooks/provider/decision"
  }
}
Response

JSON

{
  "provider_verification_id": "pv_scr_001",
  "status": "PENDING",
  "created_at": "2026-02-12T10:02:00Z"
}
4.4 Provider Decision Webhook (Inbound)
4.4.1 Webhook Endpoint
Endpoint (Onboarding): POST /webhooks/provider/decision

4.4.2 Security (HMAC)
Provider MUST send:

X-Event-Id: <unique_event_id>
X-Timestamp: <unix_epoch_seconds>
X-Signature: <hex_hmac_sha256>
Signature calculation (example policy)

Payload to sign: <timestamp>.<raw_request_body>
Signature: hex(HMAC_SHA256(webhook_secret, payload_to_sign))
Onboarding MUST:

Reject if timestamp is older than acceptable window (e.g., 5 minutes) to prevent replay
Reject if signature invalid (401/403)
Store security failure as an audit event
4.4.3 Idempotency
Use X-Event-Id as the primary idempotency key.
If X-Event-Id already processed successfully:
Return 200 OK
Do not apply state changes again
4.4.4 Webhook Body (example)
JSON

{
  "application_id": "a1b2c3d4-....",
  "provider_verification_id": "pv_kyc_001",
  "check_type": "KYC",
  "decision": "PASS",
  "risk_band": "LOW",
  "reason_codes": ["DOC_MATCH", "LIVENESS_OK"],
  "completed_at": "2026-02-12T10:10:00Z"
}
4.4.5 Webhook Response
200 OK when processed (even if application routed to manual review)
401/403 when signature validation fails
422 when payload invalid/unprocessable (still log + alert)
500 only for unexpected server errors (provider may retry)
4.4.6 Decision Aggregation Rules (multiple checks)
Onboarding typically receives decisions for multiple checks (KYC + KYB + Screening).
A final application decision is derived when:

Required checks are completed, AND
Decision rules are applied:
Final Decision Rules

If ANY required check has decision = FAIL → application = Rejected (auto-reject)
Else if ANY required check has decision = REVIEW → application = Under Manual Review
Else if all required checks are PASS:
If auto-approval eligible → Approved
Else → Under Manual Review
Onboarding MUST persist:

Raw webhook payload
Derived decision and reasons
Timestamps and reference IDs
5. CRM Integration (Lead-first)
5.1 Assumed CRM API Base
Sandbox: https://api.crm.example/v1
5.2 Authentication (assumption)
OAuth2 bearer token:
Authorization: Bearer <crm_access_token>
5.3 Object Model (Lead-first)
Create/Update Lead for every submitted application.
Convert Lead → Account + Contact on approval.
Do NOT convert on rejection.
5.4 Create Lead (on Submit)
Endpoint: POST /leads

Headers

Authorization: Bearer <token>
Idempotency-Key: APP:<application_id>:CRM:LEAD_CREATE
X-Correlation-Id: <application_id>
Body (example)

JSON

{
  "application_id": "a1b2c3d4-....",
  "customer_type": "SME",
  "lead_status": "Submitted",
  "email": "rep@corp.com",
  "phone": "+49123456789",
  "representative_first_name": "Anya",
  "representative_last_name": "Meyer",
  "company_legal_name": "Meyer Consulting GmbH",
  "company_registration_number": "HRB123456",
  "onboarding_status": "Verification In Progress"
}
Response

JSON

{
  "crm_lead_id": "00Qxx0000001234",
  "created_at": "2026-02-12T10:05:00Z"
}
5.5 Update Lead (on Status Changes)
Endpoint: PATCH /leads/{crm_lead_id}

Update when status changes to:

Verification In Progress
Under Manual Review
Awaiting Additional Info
Approved
Rejected
Include fields:

onboarding_status
provider_reference_ids (optional string list)
decision_reason_codes (when rejected)
timestamps (submitted_at/approved_at/rejected_at)
5.6 Convert Lead (on Approval)
Endpoint: POST /leads/{crm_lead_id}/convert

Body (example)

JSON

{
  "convert_on_approval": true,
  "create_opportunity": false
}
Response

JSON

{
  "crm_account_id": "001xx000000abcd",
  "crm_contact_id": "003xx000000wxyz",
  "converted_at": "2026-02-12T10:12:00Z"
}
5.7 CRM Error Handling & Queues
5.7.1 Retryable errors (retry queue)
HTTP 5xx
timeouts / network errors
HTTP 429 (rate limit) → retry after Retry-After if present
Action:

Queue retry with backoff
Mark crm_sync_status = PENDING_RETRY
Alert Ops after threshold
5.7.2 Non-retryable errors (data issue queue)
HTTP 400/422 validation errors
Action:

Mark crm_sync_status = FAILED_DATA_ISSUE
Create Ops/Admin task with error details
Do not repeatedly retry until corrected
5.8 Onboarding → CRM Field Mapping (sample)
Final field names depend on CRM configuration; below is a representative mapping.

Onboarding (logical)	CRM Field (example)	Notes
application_id	Lead.Application_ID__c	unique external key
onboarding_type	Lead.Customer_Type__c	INDIVIDUAL/SME
status	Lead.Onboarding_Status__c	mapped enum
representative.first_name	Lead.FirstName	for SME, rep contact details stored on Lead
representative.last_name	Lead.LastName	
representative.email	Lead.Email	
representative.phone	Lead.Phone	
company.legal_name	Lead.Company	or custom field
company.registration_number	Lead.Company_Reg_No__c	
decision	Lead.Onboarding_Decision__c	Approved/Rejected
reason_codes	Lead.Decision_Reason_Codes__c	comma-separated or related list
provider_verification_ids	Lead.Provider_Refs__c	optional
6. Notification Gateway (brief)
Events

Submission confirmation
Request More Info
Approved
Rejected
Requirements

Do not include restricted compliance details in applicant notifications.
Include application_id reference (masked/short form) and a safe next-step message.
7. Security & Privacy Considerations
Webhook endpoint must be protected against spoofing and replay:
HMAC signature + timestamp window + idempotency
PII must not be logged in plain text; logs must be sanitized.
Documents stored encrypted; access controlled by RBAC; access logged.
Store raw provider payload securely (restricted access).
8. Audit Requirements (integration events)
The onboarding system MUST generate audit events for:

Provider check created (request submitted)
Webhook received (including event_id, provider_verification_id)
Decision derived and applied (Approved/Manual Review/Rejected)
CRM Lead create/update/convert attempts (success/failure)
Retry queue actions and final failures
9. Testability Notes (what QA should verify)
Webhook signature validation (valid/invalid)
Idempotency (duplicate X-Event-Id ignored)
Multiple-check aggregation rules (KYC+KYB+Screening)
Provider FAIL always auto-rejects (regardless of other PASS checks)
CRM conversion occurs only after approval
CRM retry vs data-issue routing
text

---