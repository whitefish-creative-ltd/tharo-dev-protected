# Feature Spec: Password login

**ID:** LOGIN
**Parent spec:** Kestrel first-party login v1.0
**Status:** Verified
**Owner:** J. Studdart

## User story

As an admin at a customer with no identity provider
I want to sign in with an email address and a password
So that I can reach the product I've paid for without involving my employer's IT department

## Acceptance criteria

**AC1: Successful login**
- **Given** a user with a password credential in a tenant where password auth is enabled
- **When** they submit the correct email and password
- **Then** a session is created scoped to that tenant, and they land on the dashboard

**AC2: Wrong password**
- **Given** the same user
- **When** they submit the wrong password
- **Then** login fails with the generic error, no session is created, and the attempt is logged
  with outcome `failed`

**AC3: Unknown email**
- **Given** an email with no user in the addressed tenant
- **When** login is attempted
- **Then** the response is byte-identical to AC2 and the timing is within 50ms of AC2's mean

**AC4: Tenant has password auth disabled**
- **Given** a tenant with `password_auth_enabled = false`
- **When** a user of that tenant submits correct credentials
- **Then** login fails with the generic error and nothing indicates that the method is disabled

**AC5: SSO user, no password set**
- **Given** a user provisioned by SSO with no password credential
- **When** password login is attempted with any password
- **Then** the response is identical to AC2

**AC6: Rate limiting**
- **Given** 5 failed attempts against one account within 15 minutes
- **When** a 6th attempt is made with any password, correct or not
- **Then** it fails with the generic error, and the lock persists for 15 minutes from the 5th
  attempt

**AC7: Session scoping**
- **Given** a user whose email also exists in a different tenant
- **When** they log in to tenant A
- **Then** the session grants access to tenant A only, and no request carrying it can read
  tenant B data

**AC8: Redis unavailable**
- **Given** Redis is unreachable
- **When** login is attempted with correct credentials
- **Then** it returns 503 and no session exists. It does not succeed

**AC9: Nothing sensitive is logged**
- **Given** any login attempt, successful or failed, including one that throws
- **When** the logs are inspected at every level including debug
- **Then** no password appears in any form, not plaintext, not partial, not in a serialised
  request object, not in a stack trace

## Edge cases

| Scenario | Behaviour | AC |
|---|---|---|
| Correct password, locked account | Fails, generic error | AC6 |
| Email in two tenants | Scoped to the addressed tenant only | AC7 |
| Redis down | 503, fail closed | AC8 |
| Password auth disabled mid-session | Existing session survives to natural expiry | *see note* |
| Unicode in password | Accepted, NFC-normalised before hashing | AC1 |
| 1MB password submitted | Rejected at 1024 chars before hashing. Do not Argon2 a megabyte | AC2 |

> **Note on disabled mid-session:** deliberately not an AC. Sessions are checked at creation,
> not per request, and changing that is a session-infrastructure job that isn't in this spec.
> Logged as D004 so it's a known gap rather than a discovered one.

## What must never happen

- The response must never differ between wrong-password, no-such-user, locked, and
  method-disabled. Not in body, status, headers, or timing.
- A password must never reach a log, a trace, an error tracker, or an analytics event.
- A session must never carry more than one tenant.

## Out of scope

- [ ] Remember me / persistent sessions (NB6)
- [ ] MFA (NB2)
- [ ] Password strength meter in the UI (NB8)
- [ ] Social login (NB1)
- [ ] Self-registration (NB4)

## Dependencies

- Argon2id via `argon2-cffi`. Parameters from NFR1.
- Existing session service. Unmodified.
- Rate limiter, shared Redis, existing implementation.

## Notes for whoever builds this

The timing requirement in AC3 is the one people get wrong. If the user doesn't exist, the
obvious code returns early and it's fast, and that difference is an account enumeration oracle
that a script will find in an afternoon. Hash against a dummy credential on the not-found path.
Yes, it wastes the CPU. That's what it's for.

The `users` table has a nullable `password_hash` because SSO users don't have one. Null means
"no password credential", not "no password set yet". A user in the invite flow has a row and a
null hash. These are different states and the difference matters, check `invite_pending`, not
the hash.
