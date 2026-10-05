---
type: Page
title: System Overview & Event Architecture
description: Tidewell Mutual operates on a distributed, event-driven service-oriented architecture combined with targeted synchronous REST APIs.
resource: https://github.com/Nox-Demo-Org/kb-org-tw-customer-platform/blob/main/architecture/system-overview.md
tags:
- org-tw-customer-platform
- architecture
generated:
  by: nox/gemini-3.7-flash
  at: '2026-10-05T15:53:52Z'
---

# System Overview & Event Architecture

Tidewell Mutual operates on a distributed, event-driven service-oriented architecture combined with targeted synchronous REST APIs.

---

## Cross-Service Interactivity

```
  +------------------+       +-------------------+       +-------------------+
  |  policy-admin    |       | claims-management |       |  billing-service  |
  +--------+---------+       +---------+---------+       +---------+---------+
           |                           |                           |
           | policy.renewal.due        | claims.claim.settled      | billing.instalment.due
           |                           |                           | billing.payment.missed
           v                           v                           v
     ====================================================================
                         Google Cloud Pub/Sub Bus
     ====================================================================
                                       |
                                       v
                          +-------------------------+
                          |    notifications-hub    |
                          +------------+------------+
                                       | (On-demand contact lookup)
                                       v
                          +-------------------------+
                          |    customer-identity    |
                          +-------------------------+
```

---

## Enterprise Pub/Sub Conventions

All domain events routed through Google Cloud Pub/Sub conform to standard conventions:

1. **Topic Naming**: `<domain>.<entity>.<past-tense-verb>`
   - Examples: `policy.renewal.due`, `claims.claim.settled`, `billing.instalment.due`, `billing.payment.missed`, `payments.payout.sent`, `customer.profile.updated`.
2. **Subscription Naming**: `<consumer>-<topic-short-name>` (e.g. `notifications-hub` consumer subscriptions).
3. **Payload Structure**:
   - Serialized in JSON format.
   - Property keys in `snake_case`.
   - Identifier references as `string`.
   - Monetary values represented in pence as `integer`.

---

## Cross-Domain Event Subscriptions

| Event Topic | Publisher | Consumers | Purpose / Action |
|---|---|---|---|
| `policy.renewal.due` | `policy-admin` | `notifications-hub` | Triggers `renewal-notice` email dispatch. |
| `claims.claim.settled` | `claims-management` | `notifications-hub` | Triggers `claim-settled` email dispatch. |
| `claims.handler.assigned` | `claims-management` | *(None / Internal)* | Claim handler assignment event; explicitly **not** consumed by notifications hub. |
| `billing.instalment.due` | `billing-service` | `notifications-hub` | Triggers `instalment-reminder` email dispatch. |
| `billing.payment.missed` | `billing-service` | `notifications-hub` | Triggers `missed-payment` SMS alert. |
| `payments.payout.sent` | `payments-gateway` | `notifications-hub` | Triggers `payout-sent` SMS dispatch. |
| `customer.profile.updated` | `customer-identity` | `policy-admin` | Synchronizes profile modifications with active policies. (Note: `billing-service` and `notifications-hub` perform on-demand REST lookups rather than subscribing). |