---
type: Page
title: Notifications & Communications Framework
description: Customer communications are centralised in notifications-hub, which handles template rendering, subject line standardisation, and outbound channel delivery for all domain services.
resource: https://github.com/Nox-Demo-Org/kb-org-tw-customer-platform/blob/main/architecture/notifications-and-communications.md
tags:
- org-tw-customer-platform
- architecture
generated:
  by: nox/gemini-3.7-flash
  at: '2026-10-05T15:53:52Z'
---

# Notifications & Communications Framework

Customer communications are centralised in `notifications-hub`, which handles template rendering, subject line standardisation, and outbound channel delivery for all domain services.

---

## Ingestion Channels

`notifications-hub` accepts dispatch requests via two distinct pathways:

1. **Event-Driven Subscriptions**: Subscribes to Pub/Sub topics published by upstream domain engines (`policy-admin`, `claims-management`, `billing-service`, `payments-gateway`).
2. **Synchronous Direct API (`POST /v1/messages`)**: On-demand dispatch invoked directly by internal services. Accepts `to_customer_id`, `template`, and dynamic `data` payloads, returning `202 Accepted`.

---

## Customer Contact Detail Resolution

`notifications-hub` does not store local copies of customer contact details. Instead, when an event or REST request is received:
1. It extracts `customer_id` or `to_customer_id`.
2. It resolves the customer's active email or phone number on demand via `customer-identity` (`GET /v1/customers/{id}`).
3. It renders the template using payload data and passes the message to the designated channel.

---

## Template & Channel Mapping Catalog

| Template Key | Delivery Channel | Default Subject Line | Trigger Mechanism / Source |
|---|---|---|---|
| `renewal-notice` | Email | `"Your Tidewell policy renews soon"` | `policy.renewal.due` event or direct REST |
| `claim-settled` | Email | `"Your claim is settled"` | `claims.claim.settled` event or direct REST |
| `instalment-reminder` | Email | `"Your next payment"` | `billing.instalment.due` event or direct REST |
| `missed-payment` | SMS | `"We couldn't take your payment"` | `billing.payment.missed` event or direct REST |
| `payout-sent` | SMS | `"We've paid your claim"` | `payments.payout.sent` event or direct REST |

---

## Delivery Provider Configuration

- **Email Dispatch**: Internal SMTP relay at `smtp.mail.internal` with sender header `hello@tidewell.example`.
- **SMS Gateway**: Outbound SMS provider API authenticated via secret keys managed in Google Cloud Secret Manager.