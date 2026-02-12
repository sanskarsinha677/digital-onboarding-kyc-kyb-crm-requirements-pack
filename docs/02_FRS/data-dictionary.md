# Data Dictionary — Digital Onboarding (Individuals + SMEs)

## 1. Purpose
This data dictionary defines the logical data elements used by the onboarding portal and backend services for:
- Individual onboarding (KYC)
- SME onboarding (KYB + representative KYC + UBO capture)
- Document management
- Provider verification orchestration (including webhook decision processing)
- CRM integration (Lead-first model)

> Notes  
> - Field names are **logical** (system-independent) unless explicitly labeled as “CRM field”.  
> - Data types and constraints are designed for EU onboarding (GDPR + AMLD context).  
> - “Required” may be conditional based on onboarding type and workflow state.

---

## 2. Conventions
- **PII fields** are marked with 🔒.
- **Key** indicates an identifier used for correlation.
- Date/time values are assumed **UTC** in storage.
- Country codes use **ISO 3166-1 alpha-2** (e.g., DE, FR, IN).

---

## 3. Entities Overview
1. **ApplicantAccount** (login identity)
2. **Application** (onboarding case)
3. **Person** (Individual applicant / SME representative / UBO)
4. **Company** (SME business)
5. **Document**
6. **VerificationCheck** (provider request)
7. **ScreeningResult** (sanctions/PEP)
8. **DecisionHistory** (auto/manual decisions)
9. **AuditEvent**
10. **CRMMapping** (Lead/Account/Contact linkage)

---

# 4. ApplicantAccount (registration identity)
| Field | Type | Required | Constraints / Validation | Example |
|---|---|---:|---|---|
| applicant_account_id (Key) | UUID | Yes | system-generated | `6d2c...` |
| email 🔒 | string | Yes | RFC-like email format; unique | `user@example.com` |
| email_verified | boolean | Yes | default false | true |
| otp_failed_attempts | integer | Yes | >=0 | 1 |
| otp_locked_until | datetime | No | set when locked (FR-002) | `2026-02-12T10:00:00Z` |
| created_at | datetime | Yes | system timestamp |  |
| last_login_at | datetime | No |  |  |

---

# 5. Application (onboarding case)
| Field | Type | Required | Constraints / Validation | Example |
|---|---|---:|---|---|
| application_id (Key) | UUID | Yes | generated at first draft save (FR-006) | `a1b2...` |
| applicant_account_id (FK) | UUID | Yes | links to ApplicantAccount |  |
| onboarding_type | enum | Yes | `INDIVIDUAL` \| `SME` | `SME` |
| status | enum | Yes | See status list below | `Verification In Progress` |
| status_reason_code | string | No | optional internal reason | `UBO_INCOMPLETE` |
| not_eligible_auto_approval | boolean | Yes | default false; set by rules (FR-014) | true |
| duplicate_flag | boolean | Yes | default false | false |
| duplicate_match_type | enum | No | `EMAIL` \| `PHONE` \| `ID` \| `COMPANY_REG_NO` | `COMPANY_REG_NO` |
| preferred_channel | enum | No | `EMAIL` \| `SMS` | `EMAIL` |
| consent_terms | boolean | Yes | must be true at submit | true |
| consent_terms_at | datetime | No | required when consent true |  |
| consent_privacy | boolean | Yes | must be true at submit | true |
| consent_privacy_at | datetime | No | required when consent true |  |
| submitted_at | datetime | No | required when status ≥ Submitted |  |
| approved_at | datetime | No | set on approval |  |
| rejected_at | datetime | No | set on reject |  |
| cancelled_at | datetime | No | set on cancel |  |
| expires_at | datetime | No | draft expiry (e.g., +30 days) |  |
| created_at | datetime | Yes |  |  |
| updated_at | datetime | Yes |  |  |

### 5.1 Status Enumeration
- `Draft`
- `Submitted`
- `Verification In Progress`
- `Under Manual Review`
- `Awaiting Additional Info`
- `Approved`
- `Rejected`
- `Cancelled`
- `Expired`

---

# 6. Person (Individual / Representative / UBO)
A person record can represent:
- Individual applicant (KYC subject)
- SME representative (KYC subject)
- UBO (screening subject)

| Field | Type | Required | Constraints / Validation | Example |
|---|---|---:|---|---|
| person_id (Key) | UUID | Yes | system-generated |  |
| application_id (FK) | UUID | Yes | links to Application |  |
| person_role | enum | Yes | `INDIVIDUAL` \| `REPRESENTATIVE` \| `UBO` | `UBO` |
| first_name 🔒 | string | Yes | length 1–100 | `Anya` |
| last_name 🔒 | string | Yes | length 1–100 | `Meyer` |
| dob 🔒 | date | Yes | must be >= 18 for INDIVIDUAL/REP at submit (FR-009) | `1990-01-01` |
| nationality | string | Yes | ISO country code | `DE` |
| email 🔒 | string | Conditional | required for applicant/rep; optional for UBO | `rep@corp.com` |
| phone 🔒 | string | No | E.164 format | `+49123456789` |
| address_line1 🔒 | string | Yes | required for INDIVIDUAL/REP |  |
| address_line2 🔒 | string | No |  |  |
| city 🔒 | string | Yes |  | `Berlin` |
| postal_code 🔒 | string | Yes | country-specific patterns allowed | `10115` |
| country | string | Yes | ISO country code | `DE` |
| id_number 🔒 | string | No | if captured; used for duplicate detection (FR-015) |  |
| created_at | datetime | Yes |  |  |
| updated_at | datetime | Yes |  |  |

---

# 7. Company (SME only)
| Field | Type | Required | Constraints / Validation | Example |
|---|---|---:|---|---|
| company_id (Key) | UUID | Yes | system-generated |  |
| application_id (FK) | UUID | Yes | SME applications only |  |
| legal_name | string | Yes | length 1–200 | `Meyer Consulting GmbH` |
| registration_number | string | Yes | unique within system scope; used for duplicate detection | `HRB123456` |
| country_of_incorporation | string | Yes | ISO country code | `DE` |
| business_type | enum | No | e.g., `GMBH`, `LTD`, `SOLE_TRADER` | `GMBH` |
| vat_number | string | Conditional | optional/required by country/policy | `DE123456789` |
| registered_address_line1 | string | Yes |  |  |
| registered_city | string | Yes |  |  |
| registered_postal_code | string | Yes |  |  |
| registered_country | string | Yes | ISO country code | `DE` |
| created_at | datetime | Yes |  |  |
| updated_at | datetime | Yes |  |  |

---

# 8. UBO (beneficial ownership details)
UBO data is stored as person records with additional ownership fields.

| Field | Type | Required | Constraints / Validation | Example |
|---|---|---:|---|---|
| ubo_id (Key) | UUID | Yes | system-generated |  |
| person_id (FK) | UUID | Yes | links to Person where role=UBO |  |
| ownership_percentage | number | Conditional | 0–100; required if ownership_unknown_flag=false | 25.0 |
| ownership_unknown_flag | boolean | Yes | default false | false |
| ownership_notes | string | No | optional explanation if unknown |  |

### 8.1 Derived fields (per application)
| Field | Type | Required | Description |
|---|---|---:|---|
| ubo_total_ownership_percentage | number | Yes | sum of known UBO ownership% |
| ubo_ownership_complete | boolean | Yes | true when total=100 and no unknown flags |
| not_eligible_auto_approval | boolean | Yes | set true when ownership incomplete (FR-014) |

---

# 9. Document
| Field | Type | Required | Constraints / Validation | Example |
|---|---|---:|---|---|
| document_id (Key) | UUID | Yes | system-generated |  |
| application_id (FK) | UUID | Yes |  |  |
| document_type | enum | Yes | See type list | `PROOF_OF_ADDRESS` |
| owner_role | enum | Yes | `INDIVIDUAL` \| `REPRESENTATIVE` \| `UBO` \| `COMPANY` | `REPRESENTATIVE` |
| file_name | string | Yes |  | `passport.jpg` |
| file_type | enum | Yes | `PDF` \| `JPG` \| `PNG` | `JPG` |
| file_size_bytes | integer | Yes | <= configured max (default 10MB) | 204800 |
| upload_timestamp | datetime | Yes |  |  |
| document_date | date | Conditional | required for POA checks; used for recency (FR-019) | `2026-01-01` |
| av_scan_status | enum | Yes | `PENDING` \| `PASS` \| `FAIL` | `PASS` |
| storage_reference | string | Yes | encrypted object reference / URI | `s3://...` |
| version_number | integer | Yes | starts at 1; increments on replacement (FR-020) | 2 |
| replaced_document_id | UUID | No | link to previous version |  |
| created_at | datetime | Yes |  |  |

### 9.1 Document Type Enumeration (suggested)
- `IDENTITY_DOCUMENT`
- `PROOF_OF_ADDRESS`
- `COMPANY_REGISTRATION_DOCUMENT`
- `COMPANY_ADDRESS_PROOF`
- `UBO_IDENTITY_DOCUMENT` (optional, if policy requires)
- `OTHER_SUPPORTING_DOCUMENT`

---

# 10. VerificationCheck (Provider requests)
| Field | Type | Required | Constraints / Validation | Example |
|---|---|---:|---|---|
| verification_check_id (Key) | UUID | Yes | system-generated |  |
| application_id (FK) | UUID | Yes |  |  |
| check_type | enum | Yes | `KYC` \| `KYB` \| `SCREENING` | `KYC` |
| subject_role | enum | Yes | `INDIVIDUAL` \| `REPRESENTATIVE` \| `UBO` \| `COMPANY` | `COMPANY` |
| provider_name | string | Yes | e.g., “ProviderX” |  |
| provider_verification_id | string | Yes | returned by provider | `pv_123` |
| status | enum | Yes | `CREATED` \| `PENDING` \| `COMPLETED` \| `FAILED` | `PENDING` |
| decision | enum | No | `PASS` \| `REVIEW` \| `FAIL` (when completed) | `PASS` |
| risk_band | enum | No | `LOW` \| `MEDIUM` \| `HIGH` | `LOW` |
| reason_codes | string[] | No | provider reason codes | `["DOC_MATCH"]` |
| requested_at | datetime | Yes |  |  |
| completed_at | datetime | No |  |  |
| idempotency_key | string | Yes | unique per application+check type |  |
| raw_request_ref | string | No | pointer to stored request payload |  |
| raw_response_ref | string | No | pointer to stored response payload |  |

---

# 11. ScreeningResult (Sanctions/PEP)
If stored separately from VerificationCheck.

| Field | Type | Required | Constraints / Validation | Example |
|---|---|---:|---|---|
| screening_result_id (Key) | UUID | Yes |  |  |
| application_id (FK) | UUID | Yes |  |  |
| subject_person_id (FK) | UUID | Yes | rep/UBO person_id |  |
| screening_type | enum | Yes | `SANCTIONS` \| `PEP` | `SANCTIONS` |
| match_found | boolean | Yes |  | false |
| match_severity | enum | No | `LOW` \| `MEDIUM` \| `HIGH` |  |
| provider_reference_id | string | No |  |  |
| details_ref | string | No | pointer to raw payload |  |
| created_at | datetime | Yes |  |  |

---

# 12. DecisionHistory (auto + manual)
| Field | Type | Required | Constraints / Validation | Example |
|---|---|---:|---|---|
| decision_event_id (Key) | UUID | Yes |  |  |
| application_id (FK) | UUID | Yes |  |  |
| decision_source | enum | Yes | `AUTO_PROVIDER` \| `MANUAL_COMPLIANCE` | `MANUAL_COMPLIANCE` |
| decision | enum | Yes | `APPROVED` \| `REJECTED` \| `REQUEST_INFO` | `APPROVED` |
| decision_reason_code | string | Conditional | required for approve/reject | `UBO_VERIFIED` |
| decision_notes | string | No | free text (avoid sensitive details) |  |
| decided_by_user_id | string | Conditional | required for manual | `comp_analyst_1` |
| decided_at | datetime | Yes |  |  |

---

# 13. AuditEvent (immutable log)
| Field | Type | Required | Constraints / Validation | Example |
|---|---|---:|---|---|
| audit_event_id (Key) | UUID | Yes |  |  |
| application_id (FK) | UUID | No | present for application-related actions |  |
| event_type | string | Yes | e.g., `DOC_UPLOADED`, `WEBHOOK_RECEIVED` |  |
| actor_type | enum | Yes | `APPLICANT` \| `OPS` \| `COMPLIANCE` \| `SYSTEM` | `SYSTEM` |
| actor_id | string | No | user id or system account |  |
| event_timestamp | datetime | Yes |  |  |
| correlation_id | string | No | trace across services |  |
| event_payload_ref | string | No | pointer to detailed payload |  |

---

# 14. CRMMapping (Lead-first integration)
| Field | Type | Required | Constraints / Validation | Example |
|---|---|---:|---|---|
| crm_mapping_id (Key) | UUID | Yes |  |  |
| application_id (FK) | UUID | Yes | unique per application |  |
| crm_lead_id | string | Yes | returned by CRM on create | `00Qxx...` |
| crm_account_id | string | No | returned after conversion | `001xx...` |
| crm_contact_id | string | No | returned after conversion | `003xx...` |
| crm_sync_status | enum | Yes | `IN_SYNC` \| `PENDING_RETRY` \| `FAILED_DATA_ISSUE` | `IN_SYNC` |
| crm_last_sync_at | datetime | No |  |  |
| crm_last_error_code | string | No |  | `CRM_500` |
| crm_last_error_message | string | No | store sanitized message |  |

---

## 15. Data Retention & Privacy Notes (GDPR/AMLD)
- PII and documents must be retained per AML policy (assume **5 years configurable**).
- Access to documents and PII must be role-restricted and logged.
- Notifications must not expose restricted compliance details.

---

## 16. Typical Required Data by Onboarding Type (summary)
### Individual (minimum at submission)
- Person: name, DOB, nationality, address, email verified
- Consents: terms + privacy
- Documents: ID + POA (POA within 3 months)

### SME (minimum at submission)
- Company: legal name, registration number, registered address, country
- Representative: name, DOB, nationality, address, email verified
- UBOs: 1..N with minimum identity fields; ownership completeness impacts auto-approval
- Documents: rep ID + rep POA + company registration doc (+ company POA per policy)
- Screening: representative + all UBOs