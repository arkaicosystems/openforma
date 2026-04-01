# OpenFORMA — Open Form & Document Exchange Protocol
### Specification v1.1 — 2026-04-01
### Author: utm-3426 / Ark & Aico Labs
### Status: PUBLISHED — CC-BY-4.0
### Supersedes: v1.0 — naming alignment with protocol stack v4.0

---

## WHAT THIS IS

OpenFORMA is a protocol for exchanging structured form and document data
between systems in any regulated or compliance-driven industry.

In construction, it handles:
- Worker onboarding forms
- Method statements and risk assessments
- Site induction records
- Contract mandates and payment instructions
- Compliance declarations
- Snag lists and inspection records

```
OpenFORMA            = how structured documents move between systems
Construction Lexicon = the vocabulary OpenFORMA uses for construction entities
EventSpec            = the transport layer OpenFORMA payloads travel on
```

OpenFORMA is not a document format (not PDF, not DOCX).
It is a schema standard — a definition of what fields a document must contain
and how those fields are typed, validated, and exchanged.

---

## ORIGIN

Conceived: October 2024
Defined alongside EventSpec, Construction Lexicon, and MOTUS in a 72-hour protocol sprint
First spec: March 2026

---

## THE PROBLEM

Construction documents don't travel.

A method statement written in Microsoft Word cannot be validated by another system.
A RAMS (Risk Assessment Method Statement) produced on one platform
cannot be consumed, updated, or tracked by the site manager's system.
An induction form filled in on paper cannot be queried by a gateman's QR scanner.

Every document is a silo. Verification is manual. Compliance gaps are invisible.

OpenFORMA makes documents machine-readable, transferable, and verifiable.

---

## SCOPE

OpenFORMA defines schemas for four document classes:

```
FORMA-ONBOARD   — worker onboarding and registration documents
FORMA-SAFETY    — RAMS, method statements, risk assessments, COSHH
FORMA-SITE      — induction records, toolbox talk registers, snag lists
FORMA-COMMERCIAL — contracts, mandates, payment instructions, CIS declarations
```

---

## ID FORMAT

```
FORMA-{CLASS}-{NNN}

Examples:
  FORMA-ONBOARD-001   → Worker Registration Form
  FORMA-SAFETY-001    → Method Statement
  FORMA-SITE-001      → Site Induction Record
  FORMA-COMMERCIAL-001 → CIS Subcontractor Declaration
```

---

## DOCUMENT SCHEMA STRUCTURE

Every OpenFORMA document has:

```json
{
  "forma_version": "0.1",
  "forma_id": "FORMA-ONBOARD-001",
  "title": "Worker Registration",
  "schema_version": "1.0",
  "status": "draft | submitted | accepted | rejected | superseded",
  "created_at": "ISO8601",
  "submitted_by": "credi:swilliams",
  "submitted_to": "credi:site:canarywharf-2026",
  "fields": [ ],
  "signatures": [ ],
  "attachments": [ ]
}
```

### Field Definition

```json
{
  "field_id": "string",
  "label": "string",
  "type": "text | number | date | boolean | select | multi_select | lexicon_ref | file",
  "required": true,
  "value": "any",
  "validation": { }
}
```

`lexicon_ref` type allows a field to reference a Construction Lexicon code directly:

```json
{
  "field_id": "primary_trade",
  "label": "Primary Trade",
  "type": "lexicon_ref",
  "lexicon": "CON:ROLE",
  "value": "CON:ROLE-010"
}
```

---

## DOCUMENT SCHEMAS — FORMA-ONBOARD

### FORMA-ONBOARD-001 — Worker Registration

Fields required when a worker registers on a platform (credi.build, site system):

```
personal_id         — national insurance number (hashed, not stored plain)
full_name           — string
date_of_birth       — date
nationality         — ISO 3166-1 alpha-2
right_to_work       — boolean (confirmed) + document_type
emergency_contact   — name + phone
primary_trade       — lexicon_ref (CON:ROLE)
secondary_trades    — [lexicon_ref] (CON:ROLE)
cis_status          — lexicon_ref (CON:EMPL or CON:CIS)
utr_number          — string (if CIS self-employed)
bank_details        — sort_code + account_number (encrypted)
next_of_kin         — name + phone + relationship
```

### FORMA-ONBOARD-003 — Worker Professional Profile (CV)

The portable, machine-readable CV. Replaces the paper CV, agency registration form,
and ad-hoc profile page. Construction Lexicon-native. Verifiable. Worker-owned.

credi.build is the reference implementation — every worker profile IS a FORMA-ONBOARD-003 document.

```
display_name        — string (worker's chosen display name)
personal_statement  — string (max 500 chars — worker's own voice)
primary_role        — lexicon_ref (CON:ROLE) + display_label + tier
secondary_roles     — [lexicon_ref] (CON:ROLE) + display_label + tier
location            — area (string) + radius_miles + optional coordinates
availability        — status (available | unavailable | available_from) + notice_days + updated_at
years_experience    — integer
certifications      — [{ lexicon_id, display_name, description, status, awarded_date, expiry_date,
                         issuing_body, verified, verified_at }]
employment_history  — [{ role_label, employer, location, from_date, to_date, current, notable }]
competencies        — [string] (max 10 — worker-authored bullet points)
employment_type     — lexicon_ref (CON:EMPL or CON:CIS)
contact             — { phone, whatsapp?, email?, linkedin_url?, instagram_url?, tiktok_url? }
media               — { photo_url, photo_alt }
profile_url         — uri (canonical public profile URL)
platform            — string (issuing platform identifier)
eventspec_compatible — boolean (confirms QUERY-compatible)
```

**Tier field (on roles):**
Maps to CSCS card colour language — a signal every construction professional reads immediately.

| Value | CSCS equivalent | Meaning |
|---|---|---|
| `trainee` | Red card | In training, apprentice |
| `skilled` | Blue card | Qualified, working |
| `gold` | Gold card | Supervisory level |
| `senior` | Black card | Management / 15+ years |

**JSON Schema:** `schemas/FORMA-ONBOARD-003.json`


### FORMA-ONBOARD-002 — Worker Induction Declaration

Pre-site induction declaration form:

```
site_id             — string
induction_date      — date
worker_id           — string (credi ID or worker ref)
health_conditions   — boolean + free text
medication          — boolean (affects operating machinery)
alcohol_drug_policy — confirmed boolean
site_rules          — confirmed boolean
emergency_procedures — confirmed boolean
signature           — digital signature blob
witnessed_by        — string (site manager ID)
```

---

## DOCUMENT SCHEMAS — FORMA-SAFETY

### FORMA-SAFETY-001 — Method Statement

Core structure for a method statement (construction sequence document):

```
ms_title            — string
project_ref         — string
activity_description — text (what work is being done)
sequence_of_work    — [step: { order, description, risks, controls }]
plant_required      — [lexicon_ref] (CON:PLANT)
materials           — [{ material, coshh_ref, sds_available }]
ppe_required        — [{ item, standard }]
emergency_procedure — text
prepared_by         — string
approved_by         — string
review_date         — date
```

### FORMA-SAFETY-002 — Risk Assessment

```
ra_title            — string
activity            — string
hazards             — [{ hazard_id, description, likelihood, severity, risk_rating }]
controls            — [{ hazard_id, control_measure, residual_risk }]
risk_rating_method  — 3x3 | 5x5 | custom
persons_at_risk     — [workers | public | third_parties]
prepared_by         — string
approved_by         — string
review_date         — date
```

### FORMA-SAFETY-003 — COSHH Assessment

```
substance_name      — string
manufacturer        — string
sds_reference       — string
exposure_route      — [inhalation | skin | ingestion | injection]
wel_reference       — string (EH40 reference)
controls_required   — [ppe | ventilation | substitution | elimination]
first_aid_measures  — text
disposal_method     — text
```

---

## DOCUMENT SCHEMAS — FORMA-SITE

### FORMA-SITE-001 — Site Induction Record

```
site_id             — string
site_name           — string
induction_type      — full | refresher | visitor
worker_id           — string
induction_date      — date
topics_covered      — [string] (from site-defined topic list)
emergency_assembly  — string (muster point)
inducted_by         — string
worker_signature    — digital signature
inducted_by_signature — digital signature
```

### FORMA-SITE-002 — Toolbox Talk Register

```
talk_id             — string
site_id             — string
topic               — string
delivered_by        — string
date                — date
attendees           — [{ worker_id, name, signature }]
```

### FORMA-SITE-003 — Snag / Defect Record

```
snag_id             — string
site_id             — string
location            — string (floor / area / grid ref)
description         — text
raised_by           — string
raised_date         — date
assigned_to         — string (subcontractor or trade)
priority            — low | medium | high | critical
status              — open | in_progress | completed | rejected
completed_date      — date
completion_notes    — text
photographic_evidence — [file_ref]
```

---

## DOCUMENT SCHEMAS — FORMA-COMMERCIAL

### FORMA-COMMERCIAL-001 — CIS Subcontractor Declaration

```
subcontractor_name  — string
company_name        — string (if Ltd)
utr_number          — string
cis_status          — lexicon_ref (CON:CIS)
gross_payment_ref   — string (if gross status)
vat_registered      — boolean
vat_number          — string
bank_details        — sort_code + account_number
declaration_date    — date
signature           — digital signature
```

### FORMA-COMMERCIAL-002 — Labour-Only Subcontract

```
contractor          — string
subcontractor       — string
site_ref            — string
works_description   — text
rate_type           — hourly | daily | fixed | m2 | m3
rate_value          — number
rate_currency       — lexicon_ref (CON:MEAS currency)
payment_terms       — string
start_date          — date
end_date            — date (estimated)
signatures          — [contractor_sig, subcontractor_sig]
```

### FORMA-COMMERCIAL-003 — Payment Application (Interim Valuation)

```
application_number  — integer
site_ref            — string
period_from         — date
period_to           — date
gross_value_to_date — number
less_retention      — number
less_previous_certs — number
amount_due          — number
supporting_items    — [{ description, quantity, rate, total }]
submitted_by        — string
submitted_date      — date
due_date            — date
```

---

## TRANSPORT

OpenFORMA documents travel inside EventSpec payloads.

An EventSpec EVENT with type `INDUCTION_COMPLETE` references a FORMA-SITE-001 document:

```json
{
  "eventspec_version": "1.0",
  "type": "EVENT",
  "payload": {
    "event_type": "INDUCTION_COMPLETE",
    "site_id": "credi:site:canarywharf-2026",
    "worker_id": "credi:swilliams",
    "forma_ref": {
      "forma_id": "FORMA-SITE-001",
      "document_id": "induction-credi-swilliams-2026-03-21",
      "status": "accepted"
    }
  }
}
```

---

## SIGNING

OpenFORMA documents support three signature types:

```
DIGITAL_DRAW   — touchscreen / stylus drawn signature (JPEG blob, base64)
CRYPTOGRAPHIC  — Ed25519 key pair signing (OFSIG — OpenFORMA cryptographic sealing)
BIOMETRIC      — biometric device reference (future — hardware dependent)
```

For legal admissibility under UK Electronic Identification and Trust Services
(eIDAS / ETSI), CRYPTOGRAPHIC signing is the recommended standard.

---

## RELATIONSHIP TO OTHER PROTOCOLS

```
EventSpec            — transport layer for OpenFORMA payloads
Construction Lexicon — vocabulary referenced in lexicon_ref fields (CON: prefix)
MOTUS                — movement events that trigger OpenFORMA document creation
```

---

## GOVERNANCE

OpenFORMA is an open protocol. Published under Creative Commons CC-BY 4.0.
Proposals via the FieldCore Proposal (FCP) process (same as EventSpec / Construction Lexicon).

---

## ROADMAP

1. JSON Schema files for each FORMA document type
2. Construction Lexicon seed set — ROLE, PLANT, CERT, EMPL, CIS, MEAS codes
3. OFSIG verification implementation
4. Builder — drop a document, extract a schema
5. Domain Registry Index — register additional Lexicons
6. Open standard submission (BSI, W3C)

---

*OpenFORMA Spec v1.1 — 2026-04-01*
*Published under CC-BY-4.0.*
