# GuestLens

**A Consent-Based, On-Device Guest Record Continuity Assistant for Hotel Receptions**

> Submitted for the Snapdragon® AI Lab Build & Present Challenge by Qualcomm India

---

## Overview

GuestLens is a privacy-first, on-device guest record continuity assistant designed to run on a Snapdragon-powered HP PC at a hotel reception desk.

It helps hotel staff retrieve prior guest records and manage documented incidents — without cloud biometric processing, without automated service decisions, and without any data leaving the device.

---

## The Problem

Most mid-tier and budget hotels in India maintain guest records through handwritten registers, photocopied IDs, and basic PMS software. These tools record that a guest stayed — but they do not link that record to a verified face.

When a guest returns, staff have no reliable way to surface their prior record. If a guest caused damage, disputed a bill, or behaved disruptively on a prior stay, there is no mechanism to know.

CCTV exists but is not searchable by identity, not linked to guest records, and only reviewed after an incident has already occurred.

GuestLens bridges a guest's face and their record — with consent, on-device, no cloud required.

---

## What GuestLens Does

- With explicit consent, captures a face embedding at check-in and links it to the guest record
- Detects possible returning guests and surfaces prior records for staff to verify manually
- Allows staff to attach structured, manager-approved incident records to a guest file
- Allows guests to request access to, correction of, or deletion of their records
- Stores all data locally on the Snapdragon HP PC, AES-256 encrypted, with no cloud dependency
- Automatically purges biometric embeddings 180 days after checkout

## What GuestLens Does Not Do

- Store any raw face image at any point
- Make any automatic service decision
- Share any data with any external party, partner hotel, or cloud service
- Process biometric data without explicit, affirmative, informed guest consent
- Replace statutory guest registers, identity document procedures, or Form C foreign guest reporting
- Claim to be a law enforcement or criminal detection tool in any form

---

## Technical Architecture

```
Guest approaches counter
        │
        ▼
┌──────────────────────────┐
│   Consent Notice Screen   │  ← Plain language, affirmative selection required
└──────────────────────────┘
        │ Consent given
        ▼
┌──────────────────────────┐
│   3–5 Frame Capture       │  ← Webcam, controlled reception desk conditions
└──────────────────────────┘
        │
        ▼
┌──────────────────────────┐
│   Frame Quality Check     │  ← OpenCV, CPU
│   Best frame selected     │  ← All other frames deleted immediately
└──────────────────────────┘
        │
        ▼
┌──────────────────────────┐
│   Face Alignment          │  ← 5-point landmark alignment, CPU
└──────────────────────────┘
        │
        ▼
┌──────────────────────────┐
│   ArcFace Embedding       │  ← ResNet-50, INT8 quantized via QNN SDK
│   Hexagon NPU             │  ← 512-dim L2-normalized vector
│   Target: < 400ms         │  ← Selected frame deleted after embedding
└──────────────────────────┘
        │
        ▼
┌──────────────────────────┐
│   FAISS Similarity Search │  ← IndexFlatIP, cosine similarity
│   CPU                     │  ← Calibrated threshold, not universal
│   Target: < 100ms         │  ← Top result = possible match only
└──────────────────────────┘
        │
        ▼
┌──────────────────────────┐
│   Staff Review Screen     │  ← "Possible prior record — please verify"
│   Human decision only     │  ← ID + booking confirmation as normal
└──────────────────────────┘
        │
        ▼
┌──────────────────────────┐
│   Encrypted Local Storage │  ← AES-256 SQLite, TPM-backed key
│   Role-based access       │  ← Reception / Manager / Admin
│   Append-only audit log   │  ← Protected from modification
└──────────────────────────┘
```

---

## Technology Stack

| Component | Technology |
|---|---|
| Face Embedding | ArcFace ResNet-50, INT8 quantized via Qualcomm QNN SDK |
| Face Detection | YOLOv8 face variant (evaluated during testing) |
| Frame Quality Check | OpenCV |
| Similarity Search | FAISS IndexFlatIP |
| Backend | FastAPI (Python) |
| Frontend | Next.js 15 |
| Database | SQLite, AES-256 encrypted, TPM-backed key |
| NPU Execution | Snapdragon Hexagon NPU via Qualcomm AI Stack |
| Target Device | HP OmniBook X — Snapdragon X Elite |
| Phase 2 LLM | Llama 3.2 3B, quantized via Qualcomm LLM runtime |

---

## Hardware Requirements

| Requirement | Specification |
|---|---|
| Primary Device | Snapdragon-powered HP PC |
| Camera | Built-in 720p minimum / External USB 1080p recommended |
| Additional Hardware | Basic desk ring light (under ₹500) for low-light setups |
| RAM | Minimum 16GB |
| Storage | Minimum 256GB |
| OS Security | BitLocker full-disk encryption enabled (deployment prerequisite) |
| Internet | Not required for any core feature |

---

## Consent and Data Lifecycle

```
Check-in
  → Guest sees plain-language consent notice
  → Guest selects Agree or Continue Without Face Recognition
  → Consent recorded: timestamp, version, purpose, retention period
  → Camera activates only after affirmative consent

Processing
  → Capture sequence → best frame selected → all others deleted immediately
  → Face alignment → embedding generated on Hexagon NPU → frame deleted
  → Only 512-dim embedding retained, stored AES-256 encrypted

Retention
  → 180-day window from checkout date
  → Embedding auto-purged at expiry
  → Non-biometric visit summary retained under operational records purpose

Return Visit
  → Fresh consent obtained independently
  → New 180-day window begins from new checkout date

Deletion Request
  → Propagated to: database, FAISS index, encrypted backup, temp files
  → Audit entries anonymised rather than deleted
  → Deletion confirmed to guest in writing
```

---

## Responsible AI

- **No automated decisions** — system surfaces information, staff decide
- **No auto-denial** — no guest is denied service based on system output alone
- **Fairness evaluation** — ArcFace evaluated across skin tones, ages, genders, glasses, facial hair, head coverings, and indoor lighting conditions
- **Threshold calibration** — driven by acceptable false match rate, not overall accuracy alone; false match rate and false non-match rate reported separately
- **Incident record integrity** — mandatory reason category, staff attribution, manager approval, expiry dates, and guest appeal rights
- **Bias policy** — no flag may be raised based on protected characteristics or a guest's refusal of face capture

---

## Compliance Intent

GuestLens is designed with DPDP Act 2023 principles in mind — purpose limitation, storage limitation, data minimisation, consent, and data principal rights.

Actual compliance depends on hotel-specific legal review, applicable rules under the DPDP Act, hotel privacy notices, grievance redressal procedures, and the data fiduciary obligations of the hotel as the primary decision-maker for guest data.

GuestLens operates as a software tool. The hotel is the data fiduciary responsible for its lawful use.

GuestLens complements existing guest-register and foreign-guest reporting workflows. It does not replace statutory registers, identity-document procedures, signatures, or Form C reporting.

---

## MVP Demonstration Scenario

1. Guest approaches reception. Staff opens GuestLens check-in screen.
2. Guest sees plain-language consent notice. Guest selects Agree.
3. System captures 3-frame sequence. Best frame selected. All others deleted immediately.
4. Face alignment and ArcFace embedding generated on Hexagon NPU. Frame deleted.
5. Embedding stored encrypted. Guest record created with name, check-in date, consent timestamp.
6. Same guest returns. Consents again. System detects possible prior record.
7. Staff see: *"Possible returning guest — please verify using ID and booking."*
8. Staff verify manually. Prior visit history displayed.
9. Manager attaches approved incident record with mandatory reason and reference.
10. Second guest declines face capture. Check-in proceeds normally with no differential treatment.
11. Guest deletion request demonstrated — full propagation to database, index, and backup confirmed.
12. Performance metrics displayed — benchmarked on HP OmniBook X with Snapdragon X Elite.

---

## Roadmap

### Phase 1 — Current Submission Scope
- Consent-first enrollment with affirmative selection
- On-device face quality check, alignment, and embedding
- Returning guest detection with calibrated similarity threshold
- Structured incident records with manager approval, expiry, and appeal
- Guest rights interface — access, correction, deletion
- AES-256 encrypted local storage with TPM-backed key management
- Role-based access control (Reception / Manager / Admin)
- Append-only audit logging

### Phase 2 — Natural Language Query Interface
- Locally running Llama 3.2 3B quantized via Qualcomm LLM runtime
- Constrained structured query representation — not raw SQL generation
- Application validates schema and constructs all queries
- Manager-facing only, role-permission enforced

### Future Research — Not Part of Current Product Plan
- Cross-hotel record continuity, subject to independent legal review, consent framework design, governance assessment, and data protection impact assessment

---

## Repository Status

> This repository contains the full proposal and system architecture for GuestLens, submitted as part of the Snapdragon® AI Lab Build & Present Challenge.

---

## Author

**Santosh K Kammar**
GitHub: [@SKKammar](https://github.com/SKKammar)
