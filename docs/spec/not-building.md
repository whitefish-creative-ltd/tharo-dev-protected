# Not Building Register: Kestrel first-party login

**Status:** Living document, append only
**Owner:** J. Studdart
**Last updated:** 2026-04-02

## Not building

| # | Thing | Why not | Decided | Revisit when |
|---|---|---|---|---|
| NB1 | Social login (Google/Microsoft/GitHub) | Solves a problem we don't have. Our users have work email and their employer already has an IdP or doesn't. Adds three integrations and a support surface for zero tickets in Q1 | 2026-03-09 | Ever requested by a paying customer. It hasn't been |
| NB2 | MFA | Genuinely needed, genuinely not this phase. Doing it properly means enrolment, recovery codes, device management and a support process for lockouts. That's bigger than this whole spec | 2026-03-09 | Q3. Committed, not deferred indefinitely |
| NB3 | Passkeys | Same reasoning as NB2, plus the recovery story is worse. Right answer eventually | 2026-03-09 | After MFA exists |
| NB4 | Self-registration | Kestrel is B2B. An account belongs to a tenant that pays for it. Open registration would create orphan users with no tenant and no payer | 2026-03-09 | Never. This is a business model decision, not a technical one |
| NB5 | Password expiry / rotation | Contradicts NFR5 and current NCSC and NIST guidance. Forced rotation produces `Summer2026!` and a sticky note. It is security theatre with a measurable negative effect | 2026-03-12 | Never. If a security questionnaire demands it, that's a sales conversation, not an engineering one. See D002 |
| NB6 | "Remember me" / persistent sessions | 8-hour expiry is a deliberate decision inherited from SSO. A 90-day cookie is a different security posture and needs its own analysis, not a checkbox someone adds because every login form has one | 2026-03-12 | If session length is revisited properly, as its own piece of work |
| NB7 | Admin-set password policies per tenant | Configurability we'd have to support forever, to let customers make their security worse. NFR5 is the policy | 2026-03-12 | Enterprise contract requires it in writing |
| NB8 | Password strength meter | Looks helpful, mostly lies. zxcvbn-style meters rate `P@ssw0rd123` well. The breach-list check in NFR5 does the actual work. A green bar next to a breached password is worse than no bar | 2026-03-14 | Never |
| NB9 | Account merging (SSO user adopts a password) | FR6 says both methods work if both exist. It does not say we build a flow for adding one to the other. Nobody has asked | 2026-03-14 | If FR6 generates tickets |

## Overturned

| # | Thing | Why now | CR | Date |
|---|---|---|---|---|
| (none) | (none) | (none) | (none) | (none) |

Nothing yet. Note that NB6 and NB8 were both *built anyway* during the work, see the drift log
in Change Register 07. Neither is on this list because we changed our minds. They're on it
because we didn't, and they got built regardless.

That distinction is the entire point of keeping this file separate.
