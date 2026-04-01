---
document: ARCHITECTURE-PRD
version: 0.3
date: 2026-04-01
status: active — decisions locked
author: utm-3426 + Walter
supersedes: ARCHITECTURE-PRD v0.2
---

# OpenFORMA Architecture PRD
### Stack Architecture, Schema Taxonomy, Locked Decisions

---

## One-Sentence Version

OpenFORMA is the base protocol and portable record container.
FormaSchema is the schema governance layer built on it.
Every document and identity record — in any industry — is a FormaSchema schema.
EventSpec is the standalone event specification layer. The Construction Domain Registry is the construction vocabulary.
xQ Profile is the human identity render built on IDN-PRO-001.

---

## Locked Decisions

These are settled. They do not reopen without a FRAGO.

| Decision | Locked answer |
|---|---|
| FORMA-DDI-001 rename | Renamed to FIN-DOC-004. Legacy alias preserved in metadata. |
| FormaSchema brand | Internal / technical name only. Public language is "OpenFORMA schema." |
| PLT / ORG / SIT / VEH / PRD Profile names | Internal class names. Not public brand. Not final. |
| Local-first vs SaaS | Local-first. Protocol stands without the platform. SaaS after pilot validates format. |
| PWA ingestion scope | Deferred. Not until native schema flows (induction + invoice) prove in field. |
| CON domain | Removed as peer domain. Construction is an industry overlay, not a functional domain. |
| Canonical payload | JSON is the only authoritative payload. HTML is a render. YAML is optional metadata. Neither overrides JSON. |
| Signing algorithm | Ed25519 as primary. RSA-PSS as enterprise compatibility fallback. PKCS#1 v1.5 not used going forward. |
| EventSpec event schema reference | COM-DOC-003 (RAMS). All stale prefixed IDs retired. |
| OCX / OCP → EventSpec | Renamed to EventSpec — standalone event specification layer. OCX and OCP are both retired. |

---

## The Layer Model

```
┌─────────────────────────────────────────────────────────────────┐
│  OFSIG                                                          │
│  Cryptographic signing envelope. Used by all OpenFORMA records. │
│  Algorithm: Ed25519 (primary). RSA-PSS (enterprise fallback).   │
│  Canonical JSON via RFC 8785 before hashing.                    │
│  Envelope includes: key_id, issuer_id, algorithm, signed_at,   │
│  schema_id, schema_version, record_id, payload_hash.           │
│  Proves integrity. Trust is determined by issuer identity       │
│  and verifier policy — not by the signature alone.             │
└─────────────────────────┬───────────────────────────────────────┘
                          │
┌─────────────────────────▼───────────────────────────────────────┐
│  OpenFORMA                                                      │
│  Base layer. The protocol. Invisible infrastructure.            │
│                                                                 │
│  Defines:                                                       │
│  · .forma package format                                        │
│  · Canonical JSON payload (authoritative)                       │
│  · HTML render layer (derived view)                             │
│  · YAML metadata block (optional, non-authoritative)            │
│  · Chrome standard (sticky header, sidebar, page-foot QR)       │
│  · OFSIG signing envelope                                       │
│  · Exchange protocol                                            │
│                                                                 │
│  Does NOT define: what fields are in a document, what a         │
│  document IS, how documents are classified.                     │
│  That is FormaSchema.                                               │
└─────────────────────────┬───────────────────────────────────────┘
                          │
┌─────────────────────────▼───────────────────────────────────────┐
│  FormaSchema                                                        │
│  Schema registry. Built on OpenFORMA.                           │
│  The naming authority. The classification system.               │
│                                                                 │
│  Internal name. Public language = "OpenFORMA schema."           │
│                                                                 │
│  Defines two record classes:                                    │
│                                                                 │
│  DOCUMENT  →  signed, filed, immutable payload                  │
│               Point-in-time transaction or declaration          │
│               Examples: invoice, induction, DDI                 │
│               Lifecycle handled via document chain links,       │
│               not payload mutation                              │
│                                                                 │
│  PROFILE   →  living record, versioned, has an owner            │
│               Subject is an entity that changes over time       │
│               Examples: worker identity, machine asset, site    │
│               Fields carry provenance class per claim           │
│                                                                 │
│  Schema naming convention:                                      │
│  {DOMAIN}-{CLASS}-{NNN}                                     │
│                                                                 │
│  Functional domains (what the record does):                     │
│  FIN   Financial                                                │
│  OPS   Operational                                              │
│  COM   Compliance & Safety                                      │
│  HR    Human Resources                                          │
│  LEG   Legal & Commercial                                       │
│  LOG   Logistics                                                │
│  IDN   Identity & Profile                                       │
│                                                                 │
│  Industry overlays (sector vocabulary applied to schemas):      │
│  CON   Construction — uses Construction Domain Registry vocabulary                      │
│  (MED, EDU, TRANS, MFG — reserved, not yet defined)            │
└─────────────────────────┬───────────────────────────────────────┘
                          │
          ┌───────────────┴───────────────┐
          │                               │
┌─────────▼──────────┐         ┌──────────▼─────────────────────┐
│  xQ Profile        │         │  FormaSchema Schemas                │
│  IDN-PRO-001   │         │  (all industries)               │
│  Human-facing      │         │                                 │
│  render spec for   │         │  DOCUMENT + PROFILE classes     │
│  worker identity.  │         │  across all functional domains  │
│  Person only.      │         │  with optional industry overlay │
└────────────────────┘         └─────────────────────────────────┘
                                          │
                          ┌───────────────┴───────────────┐
                          │                               │
               ┌──────────▼──────────┐        ┌──────────▼──────────┐
               │  Construction       │        │  EventSpec          │
               │  Domain Registry    │        │  Standalone event   │
               │                     │        │  specification.     │
               │  Roles, certs,      │        │  Any industry.      │
               │  plant, equipment.  │        │  Emitted by any     │
               │  Referenced BY      │        │  system. Does not   │
               │  schemas. Does not  │        │  produce .forma     │
               │  produce documents. │        │  files.             │
               └─────────────────────┘        └─────────────────────┘
```

---

## The .forma Package

A `.forma` file is a package. Its contents:

| Layer | Purpose | Authority |
|---|---|---|
| `payload.json` | Canonical signed JSON | **Authoritative** |
| `render.html` | Human-readable view | Derived. Never authoritative. |
| `meta.yaml` | Optional metadata, editor hints | Non-authoritative. Informational only. |
| `ofsig.json` | Signing envelope | Cryptographic proof of payload integrity |
| `attachments/` | Supporting files (photos, docs) | Referenced from payload. Not payload. |

**Rule, no exceptions:** If HTML and JSON disagree, JSON wins. If YAML and JSON disagree, JSON wins.

---

## Canonical Payload Model

Every OpenFORMA record has this envelope structure in `payload.json`:

```json
{
  "openforma_version": "0.2",
  "schema_id": "COM-DOC-004",
  "schema_version": "1.0",
  "record_id": "uuid-v4",
  "record_class": "DOCUMENT",
  "industry_overlay": "construction",
  "issuer_id": "credi:org:apexgroundworks",
  "subject_id": "credi:worker:swilliams",
  "site_id": "credi:site:canarywharf-2026",
  "created_at": "2026-03-25T09:14:00Z",
  "effective_at": "2026-03-25T09:14:00Z",
  "status": "signed",
  "lifecycle": {
    "supersedes": null,
    "superseded_by": null,
    "expires_at": null,
    "withdrawn_at": null
  },
  "payload": { },
  "attachments": [ ],
  "signatures": [ ],
  "ofsig_ref": "ofsig.json"
}
```

---

## OFSIG Signing Envelope

`ofsig.json` structure:

```json
{
  "ofsig_version": "0.2",
  "algorithm": "Ed25519",
  "key_id": "key-uuid",
  "issuer_id": "credi:org:apexgroundworks",
  "signed_at": "2026-03-25T09:14:03Z",
  "schema_id": "COM-DOC-004",
  "schema_version": "1.0",
  "record_id": "uuid-v4",
  "payload_hash": "SHA-256 of RFC 8785 canonical JSON",
  "public_key": "Ed25519 public key, base64url",
  "signature": "Ed25519 signature, base64url",
  "trust_hint": "optional — registry URL or issuer certificate reference"
}
```

**Trust model — explicit:**
- A valid OFSIG signature proves the payload has not been altered since signing.
- It does not prove the issuer is legitimate.
- It does not prove the schema content is accurate.
- Trust is determined by: issuer identity + schema compliance + verifier acceptance policy.
- Verifiers must evaluate all three. Cryptographic validity alone is not acceptance.

---

## Lifecycle Model

DOCUMENT records are immutable at the payload level. The lifecycle chain around them is not.

| Status | Meaning |
|---|---|
| `draft` | In progress. Not yet signed. |
| `signed` | Payload locked. OFSIG applied. |
| `submitted` | Delivered to receiving party. |
| `accepted` | Receiving party has confirmed acceptance. |
| `rejected` | Receiving party has rejected. Reason logged. |
| `superseded` | Replaced by a later record. `superseded_by` field populated. |
| `withdrawn` | Retracted by issuer. `withdrawn_at` populated. Reason logged. |
| `expired` | Past `expires_at` date. System-inferred. |

**Principle:** The original payload never changes. The graph of relationships around it does.

**RAMS note:** A RAMS may be drafted, revised, and re-signed before a job starts.
Each version is a new DOCUMENT record. The chain is:
`COM-DOC-003 v1 (superseded) → COM-DOC-003 v2 (signed, accepted)`
Iteration is handled by the lifecycle chain, not payload mutation.

**Multi-signature:**
Schemas that require more than one signer (induction: worker + officer; subcontract: both parties)
declare `required_signers` and `signing_order` in their schema definition.
A document with partial signatures holds status `draft` until all required signatures are present.

---

## Versioning Model

Three independent version lines:

| Version type | Tracks | Example |
|---|---|---|
| Protocol version | OpenFORMA spec version | `openforma_version: "0.2"` |
| Schema version | Specific schema revision | `schema_version: "1.0"` |
| Record version | Instance lifecycle revision | Tracked via lifecycle chain, not field mutation |

Schema IDs are permanent. Numbers are never reused. Deprecated schemas are marked `deprecated` in the registry, not deleted.

---

## Functional Domains and Industry Overlays

These are two different axes. A schema has one functional domain and optionally one industry overlay.

**Functional domain** = what the record does
**Industry overlay** = what sector vocabulary it uses

Example:
- A RAMS is `COM-DOC-003`, functional domain = COM
- When used in construction, it carries `industry_overlay: "construction"` and references Construction Domain Registry vocabulary
- When used in a food factory, it carries `industry_overlay: "MFG"` and references a future MFG vocabulary
- The schema ID does not change. The overlay changes.

This keeps the core taxonomy clean and industry-agnostic.

---

## Extension Rules

| Field namespace | Purpose | Rules |
|---|---|---|
| Core fields | Schema-defined, required or optional | Defined in FormaSchema registry. Cannot be removed by implementer. |
| Overlay fields | Industry-specific, keyed by overlay namespace | `con.*` for construction overlay fields |
| Extension fields | Implementer-specific additions | Must use `ext.{implementer}.{field}` namespace. Ignored by other systems. |

Example extension field: `ext.credi.worker_tier = "gold"`
This does not contaminate the core schema and is ignored by non-credi systems.

---

## Profile Field Provenance

PROFILE records contain mixed-trust data from different sources.
Each field or field group carries a provenance class:

| Class | Meaning |
|---|---|
| `self_declared` | Worker / subject asserted this. Not verified. |
| `issuer_attested` | Issued by a named authority (training body, employer, CSCS). |
| `verified_copy` | Document checked by a third party. Evidence on file. |
| `platform_derived` | Calculated by the system (e.g. years from employment dates). |
| `system_inferred` | Inferred from events (e.g. cert expired = inferred from expiry date). |
| `third_party_endorsed` | Reference or endorsement from a named third party. |

This is not decoration. It is the difference between a profile that says "I have a CPCS card"
and one that says "CPCS card issuer-attested, document verified 2026-03-20, expires 2027-02-10."
The second is infrastructure. The first is a PDF.

---

## FormaSchema Schema Taxonomy

### Class: DOCUMENT
*Signed, filed, immutable payload. Point-in-time transaction or declaration.*

---

#### FIN — Financial

| Schema ID | Name | Description | Required signers |
|---|---|---|---|
| FIN-DOC-001 | Invoice | Goods or services rendered, amount due | Supplier |
| FIN-DOC-002 | Purchase Order | Authorisation to supply | Buyer |
| FIN-DOC-003 | Delivery Note | Confirmation of goods delivered | Receiver |
| FIN-DOC-004 | Direct Debit Instruction | Bank mandate for recurring payment | Account holder |
| FIN-DOC-005 | Payment Application | Contractor's claim for work completed | Contractor |
| FIN-DOC-006 | Remittance Advice | Confirmation of payment made | Payer |
| FIN-DOC-007 | Credit Note | Correction or refund against an invoice | Supplier |
| FIN-DOC-008 | Expense Claim | Worker or contractor expense submission | Claimant |

`FIN-DOC-004` — legacy alias: `FORMA-DDI-001`. Alias preserved in record metadata.

**CON overlay examples:**
- Plant hire invoice — `FIN-DOC-001` + `overlay: CON` + Construction Domain Registry-PLANT field
- CIS payment statement — `FIN-DOC-006` + `overlay: CON` + CIS deduction rate
- Subcontractor payment application — `FIN-DOC-005` + Construction Domain Registry-ROLE + site ref

---

#### OPS — Operational

| Schema ID | Name | Description | Required signers |
|---|---|---|---|
| OPS-DOC-001 | Job Sheet | Work instructions for a specific task | Supervisor |
| OPS-DOC-002 | Work Order | Formal authorisation to carry out work | Client / PM |
| OPS-DOC-003 | Site Visit Report | Record of visit, findings, actions | Inspector |
| OPS-DOC-004 | Snag List | Defects identified, remediation required | Site manager |
| OPS-DOC-005 | Prestart Check | Daily plant / vehicle / equipment inspection | Operator |
| OPS-DOC-006 | Toolbox Talk Record | Safety briefing attendance and topic | Supervisor + Attendees |
| OPS-DOC-007 | Near Miss Report | Incident near-miss logged and submitted | Reporter |
| OPS-DOC-008 | Handover Certificate | Formal handover of works to client | Contractor + Client |

---

#### COM — Compliance & Safety

| Schema ID | Name | Description | Required signers |
|---|---|---|---|
| COM-DOC-001 | Risk Assessment | Hazards identified, controls applied | Responsible person |
| COM-DOC-002 | Method Statement | Step-by-step safe working procedure | Supervisor + Site manager |
| COM-DOC-003 | RAMS | Combined risk assessment + method statement | Supervisor + Site manager |
| COM-DOC-004 | Site Induction Record | Worker inducted, rules acknowledged | Worker + Induction officer |
| COM-DOC-005 | Permit to Work | Authorisation for high-risk activity | Permit issuer + Receiver |
| COM-DOC-006 | COSHH Assessment | Hazardous substance risk assessment | COSHH officer |
| COM-DOC-007 | Fire Risk Assessment | Fire hazard assessment for premises | Assessor |
| COM-DOC-008 | LOLER Thorough Exam | Lifting equipment statutory inspection | Competent person |
| COM-DOC-009 | PUWER Inspection | Work equipment safety inspection | Competent person |

**CON overlay examples:**
- RAMS for groundworks — `COM-DOC-003` + `overlay: CON` + Construction Domain Registry-ROLE + Construction Domain Registry-SITE confined space flag
- Site induction — `COM-DOC-004` + `overlay: CON` → fires `EventSpec: INDUCTION_COMPLETE`
- Lifting operations permit — `COM-DOC-005` + Construction Domain Registry-PLANT crane ID + Construction Domain Registry-CERT appointed person

---

#### HR — Human Resources

| Schema ID | Name | Description | Required signers |
|---|---|---|---|
| HR-DOC-001 | Employment Contract | Terms of employment, role, pay | Employer + Employee |
| HR-DOC-002 | Offer Letter | Formal offer of employment | Employer |
| HR-DOC-003 | Reference | Professional reference for a worker | Referee |
| HR-DOC-004 | Timesheet | Hours worked per period | Worker + Supervisor |
| HR-DOC-005 | Disciplinary Notice | Formal written warning | HR / Manager |
| HR-DOC-006 | Absence Record | Signed absence declaration | Worker |
| HR-DOC-007 | Right to Work Declaration | Worker confirms right to work in UK | Worker |
| HR-DOC-008 | P45 / Leaver Record | Employment end record | Employer |

---

#### LEG — Legal & Commercial

| Schema ID | Name | Description | Required signers |
|---|---|---|---|
| LEG-DOC-001 | Quote / Proposal | Priced offer for works or services | Supplier |
| LEG-DOC-002 | Terms & Conditions | Standard trading terms | Supplier |
| LEG-DOC-003 | NDA | Non-disclosure agreement | Both parties |
| LEG-DOC-004 | Subcontract | Formal subcontract for works | Contractor + Sub |
| LEG-DOC-005 | LOSC | Labour-only subcontract | Contractor + Worker |
| LEG-DOC-006 | CIS Declaration | Worker declares CIS status to contractor | Worker |
| LEG-DOC-007 | Warranty Document | Product or works warranty | Provider |
| LEG-DOC-008 | Letter of Intent | Instruction to proceed pending contract | Client |

---

#### LOG — Logistics

| Schema ID | Name | Description | Required signers |
|---|---|---|---|
| LOG-DOC-001 | Goods Received Note (GRN) | Confirmation materials received | Receiver |
| LOG-DOC-002 | Waste Transfer Note | Legal record of waste removal | Carrier + Producer |
| LOG-DOC-003 | Consignment Note | Goods in transit record | Driver + Receiver |
| LOG-DOC-004 | Fuel Delivery Record | Fuel delivered to site or machine | Driver + Site manager |
| LOG-DOC-005 | Plant Hire Agreement | Machine on-hire period start | Hire desk + Client |
| LOG-DOC-006 | Off-Hire Notice | Machine return confirmed | Client + Hire desk |
| LOG-DOC-007 | Abnormal Load Permit | Oversized load movement authority | Haulier |
| LOG-DOC-008 | Skip Hire Order | Skip delivery and collection record | Supplier + Client |

---

### Class: PROFILE
*Living record. Versioned. Owner-linked. Fields carry provenance class.*

---

#### IDN — Identity & Profile

| Schema ID | Name | Entity type | Description |
|---|---|---|---|
| IDN-PRO-001 | xQ Profile | Person / Worker | Human worker identity. Xperia Quotient render. CV replacement. |
| IDN-PRO-002 | PLT Profile | Machine / Plant | Asset record. Certs (LOLER, PUWER), service history, deployment status. |
| IDN-PRO-003 | ORG Profile | Organisation | Company record. Accreditations, directors, trading history, compliance. |
| IDN-PRO-004 | SIT Profile | Site | Site record. Principal contractor, induction version, active trades, event log. |
| IDN-PRO-005 | VEH Profile | Vehicle | Commercial vehicle. MOT, insurance, tachograph, operator licence. |
| IDN-PRO-006 | PRD Profile | Product / Material | Product data. Specification, certification (CE/UKCA), traceability. |

**xQ Profile — person only.**
Same chrome. Same OFSIG. Same page-foot QR.
Different entity class = different schema. xQ does not stretch to machines, sites, or companies.

**Provenance in xQ Profile:**
Each certification carries:
- `provenance: "issuer_attested"` with `issuing_body`, `verified_at`, `document_ref`
- Or `provenance: "self_declared"` with `updated_at`

Availability status: `self_declared`. Updated by worker.
CPCS card: `issuer_attested`. Document checked, reference logged.
Employment history: `self_declared` with `consistency_checked: boolean`.

---

## Construction Domain Registry — Construction Vocabulary Overlay

**Construction Domain Registry is not a schema. It is a vocabulary.**

It provides field values for construction-domain OpenFORMA schemas.
It does not produce .forma files. It is referenced BY schemas.

```
Construction Domain Registry-ROLE-0045  = Setting Out Engineer
Construction Domain Registry-CERT-0012  = CPCS A17C Crane Supervisor
Construction Domain Registry-PLANT-0078 = Telehandler (telescopic handler, all sizes)
```

When a schema carries `industry_overlay: "construction"`, its fields may reference Construction Domain Registry codes.
When a non-construction schema carries no overlay, Construction Domain Registry codes are not used.

---

## EventSpec — Open Coordination Protocol

**EventSpec is not a schema. It is the general coordination and event protocol.**

EventSpec defines how OpenFORMA record state changes are published, routed, and consumed
across any industry. Any FormaSchema schema in any domain can emit EventSpec events.

EventSpec is industry-agnostic. It does not know what a RAMS is or what a CPCS card is.
It knows that a record changed state, who it belongs to, and where it happened.
Construction Domain Registry provides the construction vocabulary carried inside EventSpec event payloads.

When a FormaSchema document reaches a state change (signed, accepted, submitted),
it may fire an EventSpec event. The event carries a reference to the document.

```
COM-DOC-004 signed (induction)   → EventSpec EVENT: INDUCTION_COMPLETE
COM-DOC-003 signed (RAMS)        → EventSpec EVENT: RAMS_SIGNED
HR-DOC-004 submitted (timesheet) → EventSpec EVENT: TIMESHEET_SUBMITTED
IDN-PRO-001 updated (xQ cert)    → EventSpec NOTIFY: CERT_RENEWED
LOG-DOC-005 signed (plant hire)  → EventSpec EVENT: ASSET_ON_HIRE
```

**Minimum EventSpec event envelope:**

```json
{
  "eventspec_version": "1.0",
  "event_id": "EventSpec-EVT-uuid-v4",
  "event_type": "INDUCTION_COMPLETE",
  "event_class": "state_change",
  "industry_overlay": "construction",
  "occurred_at": "2026-03-25T09:14:00Z",
  "source_record_id": "EventSpec-DOC-uuid-v4",
  "source_schema_id": "COM-DOC-004",
  "source_schema_version": "1.0",
  "subject_id": "EventSpec-PSN-uuid-v4",
  "issuer_id": "EventSpec-ORG-uuid-v4",
  "site_id": "EventSpec-SIT-uuid-v4",
  "correlation_id": "optional — links related events"
}
```

**Event classes:**

| Class | Examples |
|---|---|
| `state_change` | INDUCTION_COMPLETE, RAMS_SIGNED, TIMESHEET_SUBMITTED |
| `notification` | CERT_RENEWED, CERT_EXPIRING, PROFILE_VIEWED |
| `workflow` | RAMS_ACCEPTED, RAMS_REJECTED, PERMIT_ISSUED |
| `compliance` | RIGHT_TO_WORK_EXPIRED, CERT_EXPIRED, LOLER_DUE |

EventSpec does not produce .forma files. It consumes OpenFORMA document state events.

---

## What Is Built

| Asset | Location | Status |
|---|---|---|
| OpenFORMA chrome spec | `projects/openforma/openforma-chrome.html` | Live |
| FIN-DOC-004 (legacy: FORMA-DDI-001) | `projects/openforma/forma-ddi/forma-ddi.html` | Live |
| OpenFORMA Search | `projects/openforma/openforma-search.html` | Live |
| xQ Profile v1 (IDN-PRO-001) | `projects/credi-build/clients/swilliams/ecv.html` | Live |
| Construction Domain Registry v1.1 | `github.com/arkaicosystems/construction-lexicon` | Published |
| EventSpec v1.0 | `github.com/arkaicosystems/event-spec` | Published |
| OpenFORMA Spec v0.1 | `projects/openforma/SPEC.md` | Published |

---

## What Is Next

| Priority | Schema | Why |
|---|---|---|
| 1 | COM-DOC-004 — Site Induction | Pilot jobsite in ~4 weeks. Paper → photocopy → scan → email replaced. EventSpec event trigger. |
| 2 | FIN-DOC-001 — Invoice | Universal utility. Immediate cross-industry validation. Simple enough to prove the file model clean. |
| 3 | COM-DOC-003 — RAMS | Gate integration. Compliance enforcement. Multi-signature required. |
| 4 | LEG-DOC-005 — LOSC | Replaces verbal agreements. Labour-only subcontract with CIS overlay. |
| 5 | IDN-PRO-002 — PLT Profile | Machine passport. EventSpec asset event chain. |

---

## Open Decisions

These are the only genuine remaining open questions.

| Decision | Options | What blocks it |
|---|---|---|
| DOCUMENT / PROFILE / RECORD — two classes or three? | Keep DOCUMENT + lifecycle chain vs add RECORD class for iterative docs | Needs decision before registry spec is written. Lean: keep two classes, handle RAMS iteration via lifecycle chain. |
| SaaS timeline | After induction pilot validates format in field | Unblocked by pilot result |
| PWA ingestion provenance fields | Define `source_format`, `extraction_engine`, `confidence_score`, `reviewed_by` | Not urgent. Define when ingestion spec is written. |
| xQ Profile in public spec | xQ Profile lives in credi.build — reference in architecture diagram is informational only | No action needed |

---

## What This Is Not

This document is the architecture and schema taxonomy.

It is not:
- A product PRD (user flows, success criteria, MVP scope — that is a separate doc)
- A build spec (implementation detail lives in schema files and the core SPEC.md)
- A rollout plan (pilot logistics live in the workbench)

---

*OpenFORMA Architecture PRD v0.3 · Ark & Aico Labs · 2026-04-01*
*Status: active — decisions locked*
*Supersedes: ARCHITECTURE-PRD v0.2*
