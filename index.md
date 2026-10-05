---
okf_version: '0.2'
title: Tidewell Mutual Architecture Knowledge Base
description: Welcome to the central software architecture repository for Tidewell Mutual.
generated:
  at: '2026-10-05T15:53:52Z'
---

# Tidewell Mutual Architecture Knowledge Base

Welcome to the central software architecture repository for Tidewell Mutual. This knowledge base documents shared platforms, service boundaries, cross-cutting architectural standards, and event contracts across the organisation.

---

## Core Services Catalog

| Service | Language / Stack | Domain Ownership | Primary Responsibilities |
|---|---|---|---|
| [[customer-identity]] | Go | Customer Platform | Customer authentication, ES256 JWT issuance and JWKS rotation, customer profile master data, company-wide tenure calculation. |
| [[notifications-hub]] | TypeScript / Node.js | Communications Platform | Centralised outbound multi-channel notifications (Email, SMS), template catalog, Pub/Sub event ingestion, and on-demand direct dispatch API. |
| `policy-admin` *(External/Domain)* | — | Policy Domain | Policy lifecycle management, renewal triggers (`policy.renewal.due`), subscriber to `customer.profile.updated`. |
| `claims-management` *(External/Domain)* | — | Claims Domain | Claim lifecycle and settlements (`claims.claim.settled`, `claims.handler.assigned`). |
| `billing-service` *(External/Domain)* | — | Billing Domain | Payment schedules, instalment due dates (`billing.instalment.due`), missed payment handling (`billing.payment.missed`). |
| `payments-gateway` *(External/Domain)* | — | Payments Domain | Transaction processing and claim payouts (`payments.payout.sent`). |
| `customer-portal` *(External/Client)* | — | Digital Experience | Front-end client authenticating customers via `customer-identity`. |
| `claims-intake` *(External/Client)* | — | Claims Operations | Customer-facing claim submission service consuming customer profiles. |

---

## Key Architectural Standards & Subsystems

- **[[architecture/system-overview|System Overview & Event Architecture]]**: Enterprise messaging standards, Google Cloud Pub/Sub topics, and synchronous REST dispatch flows.
- **[[architecture/identity-and-customer-master|Identity, Authentication & Customer Master]]**: Token lifecycles, asymmetric ES256 signing, JWKS local token verification, and unified tenure logic (ADR-0004).
- **[[architecture/notifications-and-communications|Notifications & Communications Framework]]**: Centralised multi-channel messaging (SMTP & SMS), template routing, and recipient contact resolution.

---

## Shared Infrastructure & Operational Patterns

- **Cloud Infrastructure**: Google Cloud Platform (GCP).
- **Event Bus**: Google Cloud Pub/Sub with standardized topic naming `<domain>.<entity>.<past-tense-verb>`.
- **Secret Management**: Google Cloud Secret Manager for rotating cryptographic signing keys and third-party SMS gateway credentials.
- **Internal Mail Relay**: Dedicated internal SMTP host (`smtp.mail.internal`) sending from `hello@tidewell.example`.

<!-- okf:contents -->

## Contents

- [Architecture](/architecture/index.md) — 3 pages.
- [Change log](/log.md) — every generation and sync, newest first.

<!-- /okf:contents -->
