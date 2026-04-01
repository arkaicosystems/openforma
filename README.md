# OpenFORMA

**Open protocol for structured, verifiable, portable records. Universal.**

OpenFORMA defines how documents and identity records move between systems. It specifies what fields a record must contain, how those fields are typed and validated, and how the record is sealed and exchanged. It is not a file format. It is a schema standard.

Published under Creative Commons CC-BY 4.0. Built by [Ark & Aico Labs](https://arkaicolabs.com).

---

## The problem

Records do not travel.

A method statement written in one system cannot be validated by another. A worker credential issued on one platform cannot be read by the site manager's system. An induction form on paper cannot be queried by a gate scanner.

Every document is a silo. Verification is manual. Compliance gaps are invisible.

This is not a construction problem. It is a universal one.

OpenFORMA makes records machine-readable, transferable, and verifiable across any industry.

---

## The stack

```
OpenFORMA          protocol layer — record container, integrity model, exchange rules
FormaSchema        schema registry — naming, classification, field contracts
EventSpec          event layer — state changes, notifications, workflow triggers
Domain Lexicons    vocabulary per industry — stable codes for domain-specific terms
```

---

## Schema taxonomy

Schemas are named by functional domain and record class.

```
{DOMAIN}-{CLASS}-{NNN}

FIN    Financial
OPS    Operational
COM    Compliance and Safety
HR     Human Resources
LEG    Legal and Commercial
LOG    Logistics
IDN    Identity and Profile

DOC    Document — signed, filed, immutable
PRO    Profile  — living record, versioned, owner-linked
```

### Published schemas

| Schema ID | Title | Status |
|---|---|---|
| IDN-PRO-001 | Worker Identity Profile | Published |
| COM-DOC-001 | Risk Assessment | Published |
| COM-DOC-002 | Method Statement | Published |
| COM-DOC-003 | RAMS | Published |
| COM-DOC-004 | Site Induction Record | Published |
| COM-DOC-005 | Permit to Work | Published |
| COM-DOC-006 | COSHH Assessment | Published |
| FIN-DOC-001 | Invoice | Published |
| FIN-DOC-004 | Direct Debit Instruction | Published |
| FIN-DOC-005 | Payment Application | Published |
| HR-DOC-001 | Employment Contract | Published |
| HR-DOC-004 | Timesheet | Published |
| LEG-DOC-004 | Subcontract | Published |
| LEG-DOC-005 | Labour-Only Subcontract | Published |
| LEG-DOC-006 | CIS Declaration | Published |
| LOG-DOC-001 | Goods Received Note | Published |
| LOG-DOC-002 | Waste Transfer Note | Published |
| OPS-DOC-001 | Job Sheet | Published |
| OPS-DOC-004 | Snag List | Published |
| OPS-DOC-005 | Prestart Check | Published |

JSON Schema files: `schemas/`

---

## Domain Lexicons

Schemas reference domain-specific vocabulary via `lexicon_ref` fields. Lexicons are maintained in the [Domain Registry Index](https://github.com/arkaicosystems/domain-registry-index).

The first registered lexicon is the Construction Lexicon (DRI-0001). It provides stable codes for construction roles, plant, certifications, site types, and employment status.

---

## Transport

OpenFORMA records travel inside [EventSpec](https://github.com/arkaicosystems/event-spec) payloads.

```json
{
  "eventspec_version": "1.0",
  "type": "EVENT",
  "payload": {
    "event_type": "INDUCTION_COMPLETE",
    "subject_id": "sub:person:uuid",
    "forma_ref": {
      "schema_id": "COM-DOC-004",
      "record_id": "uuid",
      "status": "accepted"
    }
  }
}
```

---

## Signing

Every OpenFORMA record is sealed with OFSIG. Ed25519 primary. RSA-PSS enterprise fallback. Canonical JSON via RFC 8785 before hashing.

A valid signature proves the payload has not been altered since signing. Trust is determined by issuer identity, schema compliance, and verifier policy.

---

## Reference implementation

[credi.build](https://credi.build) is the first production implementation. Every worker profile on credi.build is an IDN-PRO-001 document.

---

## Licence

Creative Commons CC-BY 4.0. Free to use, implement, and extend. Attribution required.

---

## Governance

Protocol changes via the FieldCore Proposal (FCP) process. Raise a proposal by opening an issue in this repository.

---

## Origin

Conceived October 2024. First published April 2026.
Author: [Ark & Aico Labs](https://arkaicolabs.com)
