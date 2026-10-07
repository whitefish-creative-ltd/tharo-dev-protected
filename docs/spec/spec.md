# Spec: Kestrel first-party login

**Version:** 1.0
**Status:** LOCKED
**Owner:** J. Studdart
**Last updated:** 2026-03-14

## Changelog

| Version | Date | Change | Why |
|---|---|---|---|
| 0.1 | 2026-03-09 | Initial | |
| 0.2 | 2026-03-12 | Removed social login; added §9 | Review, non-goals were implicit |
| 1.0 | 2026-03-14 | Locked | Open questions closed |

## 1. Problem

Customers without a working identity provider cannot access Kestrel, and external collaborators
cannot be invited at all. See Problem Brief 01.

## 2. Goal

Email and password authentication as an alternative to SSO, available to any tenant, without
changing anything for tenants already using SSO.

## 3. Context

- Multi-tenant Postgres. Tenant isolation by row-level security. `users` table exists and is
  currently populated only by SSO provisioning.
- Sessions are server-side, Redis-backed, 8-hour absolute expiry. Already built. Not changing.
- SSO is SAML via a third-party library. Untouched by this work.
- A `users` row may currently exist with no credential, SSO users have no password. The schema
  must tolerate both, permanently. There is no migration; both paths coexist forever.

## 4. Constraints

**Immovable**
- UK GDPR. Kestrel is data controller for auth data.
- SSO behaviour unchanged. No migration, no downtime, no visible difference.

**Expensive to move**
- Postgres and the existing session infrastructure.
- Email delivery via existing Postmark integration.

**Assumed, flagged**
- That the 8-hour session expiry is right for password users too. Inherited from the SSO
  decision, never examined for this context. Going with it; noting it. *(See D003.)*

## 5. Functional requirements

| ID | Requirement | Priority |
|---|---|---|
| FR1 | A user can authenticate with email and password | Must |
| FR2 | A tenant admin can enable or disable password auth for their tenant | Must |
| FR3 | A user can be invited by email and set a password on first use | Must |
| FR4 | A user can reset a forgotten password via emailed link | Must |
| FR5 | Failed login attempts are rate-limited per account | Must |
| FR6 | A user with both SSO and a password can use either | Should |
| FR7 | Admins can see last-login time per user | Could |

## 6. Non-functional requirements

| ID | Requirement | Measure | Priority |
|---|---|---|---|
| NFR1 | Password hashing resists offline attack | Argon2id, m=64MB t=3 p=4 | Must |
| NFR2 | Login latency | p95 < 300ms at 50rps | Must |
| NFR3 | Reset tokens are single-use and short-lived | One use, 60 min expiry | Must |
| NFR4 | Auth events are auditable | Every attempt logged with outcome, IP, timestamp | Must |
| NFR5 | Password rules follow NCSC guidance | Min 12 chars, no composition rules, breach-list checked | Must |

> NFR5 is deliberate and will get argued about. No forced special characters, no rotation, no
> complexity theatre, length plus a breach check. This follows NCSC and NIST, and it will
> still get challenged by someone's security questionnaire. D002 exists for that conversation.

## 7. Non-goals

Summary, see Not Building Register 04 for the full list and reasoning.

No social login. No MFA in this phase. No passkeys. No password expiry. No "remember me". No
account self-registration.

## 8. Edge cases and failure modes

| Scenario | Expected behaviour |
|---|---|
| Password auth disabled for tenant, user tries anyway | Generic failure. No indication the account exists or that the method is disabled |
| Email exists in another tenant | Login succeeds in the tenant being addressed only. Email is not globally unique |
| Reset requested for non-existent email | Success response. Send nothing. No enumeration |
| Reset link used twice | Second use fails, generic error |
| Reset link used after password already changed | Fails. Token invalidated on any credential change |
| Redis down | Login fails closed with a 503. No fallback to unauthenticated |
| Postmark down | Reset request returns success to the user, queues for retry, alerts ops |
| 5 failed attempts | Account locked 15 min. Same generic error as a wrong password |
| Locked account, correct password | Still fails. Lock is not a password check |
| SSO user with no password tries password login | Generic failure. Does not reveal that SSO is the method |
| Two concurrent resets | Last-issued token wins, earlier invalidated |

## 9. What must never happen

- Never log a password, in any form, at any level, including on error.
- Never log a reset token.
- Never reveal whether an email address exists, not via response body, status code, or timing.
- Never let a session cross a tenant boundary.
- Never allow password auth for a tenant that has disabled it, by any route including reset.
- Never return a different error for "wrong password" and "no such user" and "locked" and
  "disabled".

## 10. Dependencies

| Thing | Owner | If late |
|---|---|---|
| Postmark templates | Marketing | Ship with plain-text fallback |
| Breach-list API (HIBP k-anonymity) | External | Fail open on lookup error, log it. Availability of the check must not gate login |

## 11. Open questions

All closed at v1.0.

| Question | Resolution | Date |
|---|---|---|
| Build or buy? | Build. WorkOS priced at £2.8k/mo at our seat count; the requirement is narrow and the coexistence-with-SSO constraint is the hard part either way. See D001 | 2026-03-11 |
| Is the drop-off the IdP step? | Instrumented. 61% of drop-offs are at the IdP configuration screen. Confirmed | 2026-03-10 |
| Does the enterprise deal need auditor SSO? | No, email/password acceptable, they were explicit | 2026-03-05 |

## 12. Lock

- [x] Every open question closed.
- [x] Non-goals register exists and is referenced.
- [x] Every NFR has a measure.
- [x] Reviewed by someone who didn't write it. *(K. Maiden, 2026-03-13)*

**Locked by:** J. Studdart
**Date:** 2026-03-14
