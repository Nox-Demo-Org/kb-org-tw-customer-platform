---
type: Page
title: Identity, Authentication & Customer Master
description: The customer-identity service serves as the single source of truth for customer identities, master profiles, and authentication within Tidewell Mutual.
resource: https://github.com/Nox-Demo-Org/kb-org-tw-customer-platform/blob/main/architecture/identity-and-customer-master.md
tags:
- org-tw-customer-platform
- architecture
generated:
  by: nox/gemini-3.7-flash
  at: '2026-10-05T15:53:52Z'
---

# Identity, Authentication & Customer Master

The `customer-identity` service serves as the single source of truth for customer identities, master profiles, and authentication within Tidewell Mutual.

---

## Authentication and Token Architecture

Authentication uses asymmetric ES256 JSON Web Tokens (JWTs) to decouple token issuance from token verification across Tidewell services.

### Token Lifecycle
- **Access Token TTL**: 15 minutes.
- **Refresh Token TTL**: 30 days.
- **Token Exchange**: `POST /v1/auth/token` accepts customer credentials (`email`, `password`) and issues token pairs.
- **Key Rotation & Verification**: Token signing keys are rotated every 30 days in Google Cloud Secret Manager. The active public key set is exposed via `GET /.well-known/jwks.json`, enabling downstream services and API gateways to verify access tokens locally without network roundtrips.

---

## Master Customer Profile

Customer records are maintained in the central `customers` datastore and exposed via `GET /v1/customers/{id}`.

### Schema Attributes
- `id` (`string`): Unique identifier (e.g. `cust_12345`).
- `name` (`string`): Full customer name.
- `email` (`string`): Email address.
- `phone` (`string`): Contact telephone number (e.g. `+447700900077`).
- `postcode` (`string`): Postal code.
- `customer_since` (`string` RFC 3339): Initial policy inception date.
- `tenure_years` (`integer`): Calculated whole years of customer tenure.

---

## Tenure Calculation Standard (ADR-0004)

To ensure consistency across pricing, marketing, and claims services, customer tenure is computed centrally within `customer-identity`:
- **Rule**: Policy gaps under **90 days** do not reset customer tenure.
- **Resolution**: Prevents discrepancies where separate subsystems (e.g. billing, policy renewals, contact centres) calculated disparate customer loyalty metrics.