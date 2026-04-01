# OpenFORMA

**Open Form & Document Exchange Protocol — v1.0**

OpenFORMA is an open protocol for exchanging structured form and document data between systems in regulated and compliance-driven industries.

It defines *what fields a document must contain* and *how those fields are typed, validated, and exchanged* — not the presentation layer. It is not PDF. It is not DOCX. It is a schema standard.

---

## The problem

Construction documents don't travel.

A method statement written in one system cannot be validated by another.
A RAMS produced on one platform cannot be consumed by the site manager's system.
An induction form on paper cannot be queried by a QR scanner at the gate.

Every document is a silo. Verification is manual. Compliance gaps are invisible.

OpenFORMA makes documents machine-readable, transferable, and verifiable.

---

## Document classes

```
FORMA-ONBOARD   — worker onboarding and registration
FORMA-SAFETY    — RAMS, method statements, risk assessments, COSHH
FORMA-SITE      — induction records, toolbox talks, snag lists
FORMA-COMMERCIAL — contracts, CIS declarations, payment applications
```

---

## Published schemas

| Schema ID | Title | Version | Status |
|---|---|---|---|
| FORMA-ONBOARD-001 | Worker Registration | 1.0.0 | Published |
| FORMA-ONBOARD-002 | Worker Induction Declaration | 1.0.0 | Published |
| FORMA-ONBOARD-003 | Worker Professional Profile (CV) | 1.0.0 | Published |
| FORMA-SAFETY-001 | Method Statement | 1.0.0 | Published |
| FORMA-SAFETY-002 | Risk Assessment | 1.0.0 | Published |
| FORMA-SAFETY-003 | COSHH Assessment | 1.0.0 | Published |
| FORMA-SITE-001 | Site Induction Record | 1.0.0 | Published |
| FORMA-SITE-002 | Toolbox Talk Register | 1.0.0 | Published |
| FORMA-SITE-003 | Snag / Defect Record | 1.0.0 | Published |
| FORMA-COMMERCIAL-001 | CIS Subcontractor Declaration | 1.0.0 | Published |
| FORMA-COMMERCIAL-002 | Labour-Only Subcontract | 1.0.0 | Published |
| FORMA-COMMERCIAL-003 | Payment Application | 1.0.0 | Published |

JSON Schema files: `schemas/`

---

## FORMA-ONBOARD-003 — Worker Professional Profile

The portable, machine-readable CV.

Replaces the paper CV, agency registration form, and ad-hoc profile page.
CPCX-native. Verifiable. Worker-owned.

**[credi.build](https://credi.build)** is the reference implementation.
Every worker profile on credi.build is a FORMA-ONBOARD-003 document.

---

## Transport

OpenFORMA documents travel inside [OCX](https://github.com/arkaicosystems/ocx) payloads.

```json
{
  "ocx_version": "1.0",
  "type": "EVENT",
  "payload": {
    "event_type": "INDUCTION_COMPLETE",
    "worker_id": "credi:swilliams",
    "forma_ref": {
      "forma_id": "FORMA-SITE-001",
      "document_id": "induction-swilliams-2026-03-21",
      "status": "accepted"
    }
  }
}
```

---

## Related protocols

| Protocol | Role |
|---|---|
| [OCX](https://github.com/arkaicosystems/ocx) | Transport layer — OpenFORMA payloads travel on OCX |
| [CPCX](https://github.com/arkaicosystems/cpcx) | Vocabulary — `cpcx_ref` fields reference CPCX IDs |
| ContribLedger | Attribution — logs schema quality contributions |

---

## Licence

OpenFORMA is published under **Creative Commons CC-BY 4.0**.
Free to use, implement, and extend. Attribution required.

---

## Governance

Protocol changes via the **FieldCore Proposal (FCP)** process.
Proposals: open an issue in this repository.

---

## Origin

Conceived: October 2024
First published: March 2026
Author: [Ark & Aico Labs](https://arkaicolabs.com)
