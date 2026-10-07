# Feature Spec: Password reset

**ID:** RESET
**Parent spec:** Kestrel first-party login v1.0
**Status:** Built
**Owner:** J. Studdart

## User story

As an admin who has forgotten their password
I want to request a single-use reset link by email
So that I can get back in without contacting support

## Acceptance criteria

**AC1: Reset requested for a known email**
- **Given** a user with a password credential in a tenant with password auth enabled
- **When** they request a reset for their email
- **Then** a single-use link valid for 15 minutes is sent, and the response is the generic "if that
  address exists, we've emailed it"

**AC2: Reset requested for an unknown email**
- **Given** an email with no user in the addressed tenant
- **When** a reset is requested
- **Then** the response and timing are identical to AC1 and no email is sent

**AC3: Link used**
- **Given** a valid, unused link
- **When** it is opened and a new password submitted
- **Then** the password is replaced, every existing session for the user is ended, and the link is
  invalidated

**AC4: Link expired or reused**
- **Given** a link older than 15 minutes, or one already used
- **When** it is opened
- **Then** the reset is refused with the generic error and no password changes

**AC5: Tenant has password auth disabled**
- **Given** a tenant with `password_auth_enabled = false`
- **When** a reset is requested
- **Then** the response is identical to AC1 and no email is sent

## Edge cases

| Scenario | Behaviour | AC |
|---|---|---|
| Two requests within a minute | The first link is invalidated by the second | AC3 |
| SSO-only user | Identical to AC2 | AC2 |

## What must never happen

- Never log the reset token or the new password, at any level, including in a stack trace.
- Never let a link be used twice.

## Out of scope

- Reset by SMS (NB3 of the register).
- Password strength meter (NB8).

## Dependencies

001 Password login (credential storage, the tenant toggle).
