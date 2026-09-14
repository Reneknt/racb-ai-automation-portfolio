# RACBCONSULTING MVP

> **Production-deployed executive assessment and proposal platform with separated consultant/prospect authorization, assessment-bound workflow controls, deterministic proposal generation, and an operational scheduling-to-CRM handoff.**

| Attribute | Verified status |
|---|---|
| Evidence level | **Verified Implementation + Local Validation + Production Deployment Verification** |
| Implementation repository | Private |
| Remediation baseline | `79ec4df` — MVP Remediation Sprint 001 |
| Primary capability | Application Engineering |
| Secondary capabilities | Workflow Automation & Integration · Technical Operations · Security Remediation |
| Frontend | Vite + vanilla JavaScript SPA |
| Backend | Node.js + Express |
| Persistence | PostgreSQL |
| Deployment model | Static frontend hosting + containerized backend behind HTTPS reverse proxy |
| Production services | `mvp.racbconsulting.com` · `api.racbconsulting.com` · `schedule.racbconsulting.com` |

## Overview

RACBCONSULTING MVP is a production-deployed executive assessment and proposal application used to guide a prospect through a bilingual discovery workflow, persist assessment data, generate a deterministic proposal and PDF, capture controlled prospect decisions, and provide an authenticated consultant review surface.

The implementation repository remains private. This page publishes sanitized architecture, remediation scope, validation results, operational integration boundaries, and deployment evidence without exposing credentials, prospect data, private infrastructure details, or proprietary source code.

The current public case is anchored to **MVP Remediation Sprint 001**, completed and deployed at commit `79ec4df`.

## Business workflow

```mermaid
flowchart LR
    P[Prospect] --> M[RACBCONSULTING MVP]
    M --> A[Assessment + Proposal]
    A --> D{Prospect Decision}
    A --> S[Book Executive Review]
    S --> SCH[schedule.racbconsulting.com]
    SCH --> N[Existing n8n Workflow]
    N --> T[Twenty CRM Lead]
    C[Consultant] --> CP[Authenticated Consultant Portal]
    CP --> M
```

The scheduling-to-CRM automation is an existing operational integration outside the MVP repository:

**MVP → `schedule.racbconsulting.com` → prospect books → existing n8n workflow → lead created in Twenty CRM.**

The MVP does **not** implement a direct Twenty CRM integration, and the repository does not contain the external n8n workflow. The public evidence therefore treats application implementation and external operational integration as separate evidence classes.

## Application architecture

```mermaid
flowchart TD
    B[Browser] --> F[Vite SPA]
    F -->|HTTPS REST| API[Express API]
    API --> PG[(PostgreSQL)]
    API --> PDF[PDFKit Proposal PDF]
    API --> MAIL[Optional SMTP Notifications]
    API --> AUTH[Server-side Authorization]
    F --> SCH[External Scheduling Service]
```

### Frontend responsibilities

- bilingual guided assessment workflow;
- proposal presentation;
- consultant-facing review views;
- deterministic calculations mirrored against backend rules;
- public build-time configuration for API and scheduling URLs;
- prospect authorization token retained in memory rather than browser storage.

### Backend responsibilities

- assessment persistence through PostgreSQL;
- request validation and mass-assignment controls;
- consultant authentication and signed session tokens;
- assessment-bound prospect authorization;
- proposal state transitions;
- deterministic server-side report generation;
- in-memory PDF generation;
- optional SMTP notifications;
- CORS, security headers, and rate limiting.

## Remediation Sprint 001

Sprint 001 was a focused security and correctness remediation rather than a product-expansion sprint.

Verified remediation included:

- consultant and prospect authorization separated by role and authority;
- consultant passcode moved to server-side configuration;
- signed consultant sessions introduced;
- prospect proposal tokens separated from consultant tokens;
- prospect tokens bound to a single assessment;
- prospect action endpoints restricted to the assessment-bound prospect token;
- consultant sessions prevented from recording a decision as the prospect;
- proposal PDF access protected by consultant or assessment-bound prospect authorization;
- assessment UUID knowledge alone no longer authorizes reads or workflow actions;
- request validation and allowed-field controls hardened;
- calculation/report parity covered by tests;
- production scheduling URL configured as a public frontend build value;
- deployment documentation corrected to distinguish repository-verifiable facts from operator-managed infrastructure.

A PostgreSQL credential exposed during remediation operations was subsequently rotated, and the backend was recreated with the new credential. A post-rotation production query to `/api/assessments` returned HTTP 200.

A pre-remediation backend image snapshot was retained as an explicit rollback artifact:

`racb-backend:pre-remediation-2026-09-13`

## Authorization model

The remediated authorization design separates **capability** from **authority**.

```mermaid
flowchart TD
    L[Consultant Login] --> CT[Consultant Token]
    CT --> CR[Consultant Routes]

    S[Assessment Submission] --> PT[Prospect Token]
    PT --> BIND[Bound to One Assessment]
    BIND --> PA[Prospect Proposal Actions]

    CT -. rejected .-> PA
    PT -. rejected .-> CR
```

### Consultant authority

A consultant session can access consultant-only assessment review functionality and protected PDF retrieval. The consultant passcode is evaluated server-side and is not shipped to the frontend.

### Prospect authority

A successful assessment submission issues a short-lived signed token bound to that assessment. That token authorizes only the proposal workflow for the associated assessment.

The prospect token does not grant consultant access and cannot authorize another assessment.

### Deliberate boundary

The architecture does not treat possession of an assessment identifier as authorization.

This distinction is central to the remediation:

**Identifier ≠ Credential. Capability ≠ Authority.**

## Proposal generation

The MVP proposal system is deterministic rather than LLM-generated.

Report and proposal content is produced from application templates, prospect-provided discovery data, and explicit arithmetic/planning rules. Values that cannot be calculated from supplied data are represented as not calculated rather than replaced with invented fallback metrics.

The implementation also distinguishes measured/calculated findings from proposed approaches, planning estimates, and potential outcomes.

This prevents presentation language from silently turning assumptions into verified business findings.

## Scheduling and CRM integration boundary

The MVP provides a booking handoff to `schedule.racbconsulting.com`.

The current user interface opens the scheduling service; the MVP itself does not create or confirm an appointment in the application database.

After an actual booking, a separate existing n8n workflow processes the scheduling event and creates the lead in Twenty CRM.

This boundary is intentionally documented because:

- the scheduling service is external to the MVP repository;
- the n8n workflow is not duplicated inside the MVP;
- Twenty CRM integration is not falsely attributed to application source code;
- operational automation can be evidenced without misrepresenting repository ownership.

## Validation evidence

### Backend

**66 / 66 backend tests passed.**

The backend validation suite covers API behavior, authentication, authorization, request validation, calculations, email behavior, PDF behavior, and frontend/backend calculation parity using the repository's test harness.

### Frontend

**27 / 27 frontend tests passed.**

The frontend suite covers areas including escaping, configuration, calculations, and source-level guards.

### Build and E2E

The production frontend build and remediation E2E validation completed successfully before deployment.

### Production verification

After deployment:

- the remediated backend was running from the new containerized deployment;
- the remediated frontend from commit `79ec4df` was deployed to static hosting;
- consultant login was verified in production;
- frontend behavior was verified in production;
- the PostgreSQL credential was rotated after exposure during remediation;
- the backend was recreated with the rotated credential;
- a real authenticated `/api/assessments` request returned HTTP 200 after credential rotation.

These checks establish deployment verification for the remediated baseline. They do not imply exhaustive production certification of every external dependency or every future runtime condition.

## Implementation vs. validation

This case deliberately separates what exists from what was independently exercised.

| Evidence class | What it establishes |
|---|---|
| Verified implementation | Source and configuration in the private repository represent the documented application and authorization design |
| Local validation | Automated tests and build checks exercised repository behavior locally |
| Production deployment verification | The remediated frontend/backend baseline was deployed and selected production behavior was exercised successfully |
| Operational integration | The live scheduling service hands booked appointments to an existing n8n workflow that creates leads in Twenty CRM |
| Future work | Lead Identity + Assessment Lifecycle improvements are planned for Sprint 002 and are not represented as current capability |

**Implementation ≠ Validation. Deployment ≠ exhaustive certification.**

## Verified technology profile

| Area | Implementation evidence |
|---|---|
| Frontend | Vite + vanilla JavaScript SPA |
| Backend runtime | Node.js |
| API framework | Express |
| Database | PostgreSQL via `pg` |
| PDF generation | PDFKit |
| Email | Nodemailer / SMTP, optional |
| Authentication | Server-side passcode + HMAC-signed tokens |
| Prospect authorization | Short-lived signed assessment-bound token |
| HTTP controls | CORS, security headers, rate limiting, validation |
| Backend deployment | Docker container on RACBCONSULTING VPS |
| Reverse proxy / TLS | Caddy-operated production edge |
| Frontend deployment | Static hosting / cPanel |
| Scheduling | External scheduling service |
| Downstream automation | Existing n8n workflow |
| CRM | Twenty CRM |
| Version control | Git / private GitHub implementation repository |

## Capabilities demonstrated

This project provides evidence of capability in:

- full-stack application remediation;
- REST API engineering;
- browser-to-API authorization design;
- role/authority separation;
- assessment-bound signed authorization tokens;
- PostgreSQL-backed application persistence;
- deterministic business-rule implementation;
- server-side PDF generation;
- security-oriented request validation;
- Dockerized backend deployment;
- static frontend deployment;
- production credential rotation and service recreation;
- rollback preservation;
- automated backend/frontend validation;
- production verification after deployment;
- integration-boundary documentation across application, scheduler, n8n, and CRM systems.

## Deliberate boundaries

The following are **not** claimed as current MVP capabilities:

- direct MVP-to-Twenty CRM integration;
- n8n workflow ownership inside the MVP repository;
- automatic appointment creation by the MVP itself;
- AI/LLM-generated proposal analysis;
- complete CI/CD automation;
- exhaustive production security certification;
- per-consultant identity/accounts;
- server-side token revocation;
- complete assessment lifecycle recovery for abandoned or incomplete prospect sessions.

## Sprint 002 — deferred improvement

A future remediation/product sprint has been identified for **Lead Identity + Assessment Lifecycle**.

The intended direction is that a prospect should not begin an assessment without a usable identity anchor such as name plus valid email and/or phone, and incomplete assessments should preserve lifecycle state so they can be resumed or deliberately closed.

This is **planned work only**. It was intentionally excluded from Sprint 001 so that the completed remediation could be validated, deployed, and documented without uncontrolled scope expansion.

## Evidence classification

**VERIFIED IMPLEMENTATION** — application source, authorization logic, validation, deterministic calculations, PDF generation, deployment configuration, and documentation are represented in the private implementation repository at the remediated baseline.

**LOCAL VALIDATION** — 66/66 backend tests, 27/27 frontend tests, frontend build, and remediation E2E checks were completed successfully.

**PRODUCTION DEPLOYMENT VERIFICATION** — the remediated backend and frontend were deployed; consultant login and frontend behavior were verified; the database credential was rotated and the backend recreated; a post-rotation authenticated assessments query returned HTTP 200.

**OPERATIONAL INTEGRATION** — the production scheduling service feeds booked appointments into an existing n8n workflow that creates leads in Twenty CRM. This automation is intentionally documented as an external operational integration rather than source code contained in the MVP repository.

**PLANNED** — Lead Identity + Assessment Lifecycle belongs to Sprint 002 and is not included in the current verified baseline.

## Disclosure boundary

The implementation repository is intentionally private.

This Technical Evidence Center does not publish:

- production credentials or secrets;
- database passwords;
- consultant passcodes or signing secrets;
- prospect or customer PII;
- private assessment records;
- sensitive infrastructure coordinates;
- complete proprietary source code;
- private n8n workflow credentials or CRM data.

Public architecture and evidence are sanitized from verified implementation and deployment evidence.

## Interested in this architecture?

RACBCONSULTING MVP demonstrates the application-engineering side of RACBCONSULTING's broader operations architecture: governed intake, deterministic decision support, controlled authorization, scheduling handoff, automation, and CRM continuity across multiple systems.

Organizations evaluating technical operations, AI-enabled business systems, workflow automation, application remediation, or governed integration architecture may contact RACBCONSULTING for implementation or technical review work.

---

**Rene Canete / RACBCONSULTING**  
Technical Evidence Center
