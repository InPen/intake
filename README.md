# Client Intake via Docassemble (QR-Based Legal Aid Intake)

## Overview

This project implements a public, mobile-friendly client intake workflow using Docassemble, designed for legal aid environments.

Clients or staff can scan a QR code to access a Docassemble interview that:

* collects structured intake information,
* generates a court-ready PDF intake form, and
* submits the intake to the organization (initially via email, later via LegalServer API integration).

The long-term goal is to reduce manual intake friction, improve data quality, and support event-based, clinic-based, and community outreach intake workflows.

---

## Problem Statement

Traditional legal aid intake often relies on:

* paper forms,
* handwritten notes,
* duplicated data entry,
* delayed uploads into case management systems (e.g., LegalServer).

This creates:

* inconsistent data,
* lost context,
* staff time spent retyping,
* barriers for clients with limited time, mobility, or language access needs.

This project addresses those issues by providing:

* a single source of intake truth,
* a public, QR-accessible intake entry point, and
* a clear path toward direct LegalServer integration.

---

## Goals

### Phase 1 (Current)

* Build a public Docassemble intake interview
* Match an existing paper intake form exactly
* Generate a PDF intake document
* Deliver completed intakes via email or server storage
* Be usable on mobile devices(phones/tablets)

### Phase 2 (Planned)

* Post completed intakes to LegalServer:

  * as a case note on an existing matter, or
  * as a new client + new matter
* Preserve the PDF as an attachment
* Map structured fields cleanly into LegalServer fields

---

## Non-Goals (for now)

* Authentication or client accounts
* Automated eligibility determinations
* Immediate LegalServer API calls from the public interview
* Payments or document uploads

These are intentionally deferred to keep the initial deployment safe and low-friction.

---

## High-Level Architecture

```
Client / Staff Phone
        |
        |  (QR Code)
        v
Public Docassemble Interview
        |
        |  (Structured Data + PDF)
        v
Submission Handler
   ├─ Email intake inbox   (Phase 1)
   └─ LegalServer API      (Phase 2)
```

### Key Design Choice

LegalServer credentials and API tokens will never live in the public Docassemble interview.
A secure backend “bridge” will be introduced in Phase 2.

---

## Repository Structure

```
docassemble-intake/
├── questions/
│   └── client_intake.yml          # Main Docassemble interview
├── templates/
│   └── client_intake_template.docx # PDF template (Jinja2)
├── notes/
│   ├── field_map.md               # YAML ↔ DOCX variable mapping
│   ├── qr_flow.md                 # QR deployment notes
│   └── submit_behavior.md         # Submission logic notes
└── README.md
```

---

## Docassemble Interview (`client_intake.yml`)

The interview is structured into clear, maintainable sections:

1. Introduction & context
2. Client identity & metadata
3. Contact information
4. Address
5. Income
6. Rent
7. Household members (repeatable list)
8. Landlord & subsidy
9. Conditions issues
10. Additional case information
11. Meeting & attorney information
12. Problem statement
13. Attorney notes
14. Outcome
15. Review screen
16. Submit hook
17. Confirmation screen

### Design Principles

* One screen = one concept
* Explicit IDs for each screen
* Repeatable household logic isolated
* All submission logic centralized in one event hook

---

## DOCX Template (`client_intake_template.docx`)

The DOCX file defines the official intake document that will be saved, shared, and eventually attached to LegalServer.

### Template Rules

* Uses Jinja2 syntax:

  * `{{ variable }}` for fields
  * `{% if %}`, `{% for %}` for logic
* Loop and conditional tags each live on their own paragraph
* Designed for:

  * readability,
  * white space,
  * court-friendliness,
  * printing if necessary

The template mirrors the original paper intake exactly, ensuring continuity with existing workflows.

---

## QR Code Workflow

The public interview is accessed via a URL of the form:

```
https://<DOCASSEMBLE_SERVER>/interview?i=docassemble.<PACKAGE>:data/questions/client_intake.yml
```

This URL can be encoded into a QR code and placed on:

* clinic flyers,
* intake desks,
* tablets at outreach events,
* staff lanyards or cards.

---

## Submission Handling

### Current (Phase 1)

On submission:

* the intake is assembled into a PDF
* a summary + PDF are delivered via email or
* saved server-side for staff retrieval

This requires no external API permissions.

### Planned (Phase 2)

Introduce a secure backend service to:

* receive structured intake data
* authenticate with LegalServer
* create:

  * a case note on an existing matter, or
  * a new client + matter
* attach the intake PDF

---

## Security Considerations

* The intake is public by design
* No credentials or tokens are stored in the interview
* LegalServer integration will occur only server-to-server
* Personally identifiable information is handled according to organizational policies

---

## Deployment Notes

* Requires Docassemble developer access
* Email submission depends on SMTP or SendGrid configuration
* Interview must be marked public if used via QR
* PDF template filename must match YAML exactly

---

## Future Enhancements

* Bilingual intake (English/Spanish)
* Validation rules for phone/ZIP
* Admin-only QR codes tied to existing matters
* Intake dashboards
* Automated LegalServer field mapping
* Analytics on intake volume and sources

---

## Why This Matters

This project is intentionally small, focused, and extensible.

It creates:

* immediate value for staff and clients,
* cleaner data at intake,
* a foundation for deeper LegalServer integration,
* and a replicable model for other legal aid units.

---

## Author / Maintainer

- Maria Ines Folta
- Housing Paralegal | Legal Technologist
- Greater Boston Legal Services

---
