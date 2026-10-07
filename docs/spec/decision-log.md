# Decision Log: Kestrel first-party login

**Status:** Living document, append only.

---

## D001: Build first-party auth rather than adopt an identity provider

**Date:** 2026-03-11
**Status:** Accepted
**Deciders:** JS, K. Maiden

### Context

The problem brief assumed we'd build. Nobody had checked, and that assumption was carrying the
whole spec, so it got tested before locking.

### Options considered

| Option | Pros | Cons |
|---|---|---|
| WorkOS | Handles password, social, MFA, passkeys. Someone else's on-call | £2.8k/mo at current seats. The hard part, coexisting with our existing SAML without touching it, is still ours |
| Auth0 | Mature, well documented | Pricier at our shape. Migration of existing SSO users implied, which violates an immovable constraint |
| Build | Narrow requirement. No per-seat cost. Full control of the coexistence problem | We own auth forever, including the parts we said we wouldn't build yet |

### Decision

Build. The requirement is genuinely narrow (email, password, reset, invite) and every buy
option still leaves us solving the SSO-coexistence problem ourselves, which is the actual
difficulty. We'd be paying £34k/year to not solve the easy half.

### Consequences

**Easy:** no vendor, no per-seat cost, no migration, SSO untouched.

**Hard:** we own password security forever. Every future auth requirement, MFA, passkeys, is
now ours to build, and NB2 and NB3 have just become promises we made to ourselves.

**Uncomfortable:** if we get NFR1 or AC9 wrong, that's a breach with our name on it and no
vendor to share it with. That's the real price and it isn't on the invoice.

### Revisit when

MFA and passkeys both land on the roadmap in the same quarter. At that point the build cost
crosses the licence cost and the calculation inverts.

---

## D002: No composition rules, no rotation. Length plus breach check.

**Date:** 2026-03-12
**Status:** Accepted
**Deciders:** JS

### Context

Someone will ask for a special character. Then a capital. Then 90-day expiry. Each request will
sound reasonable and come from someone senior who read a compliance PDF. This entry exists so
that conversation happens once.

### Options considered

| Option | Pros | Cons |
|---|---|---|
| Composition rules + rotation | Passes security questionnaires unread. Familiar | Contradicts NCSC and NIST. Demonstrably produces worse passwords: predictable substitutions, reuse, sticky notes |
| Length + breach check | Current guidance. Actually works | Will fail a questionnaire someone didn't read properly |

### Decision

Minimum 12 characters, no composition rules, no expiry, checked against HIBP by k-anonymity.
Per NFR5.

### Consequences

We will lose an argument with a procurement checklist at some point. When that happens it is a
sales conversation about why our answer is better, not an engineering ticket. Whoever's holding
this in six months: the answer is no, and this entry is why.

### Revisit when

NCSC guidance changes. Not when someone's questionnaire disagrees with it.

---

## D003: 8-hour session expiry retained for password users, unexamined

**Date:** 2026-03-14
**Status:** Accepted
**Deciders:** JS

### Context

The 8-hour absolute expiry was chosen for SSO users, where the IdP handles re-auth and the
friction is near zero. Password users have no IdP, so every expiry is a real re-type. Nobody
examined whether the number still fits.

### Decision

Keep 8 hours. Ship. Instrument re-auth frequency and revisit with data.

### Consequences

We may be shipping a mildly annoying experience for exactly the users this project is meant to
help. Logged rather than solved, deliberately, because guessing a better number is not better than
knowing this one is unexamined. It also means the first person to suggest "remember me" has a
real argument, and NB6 is the answer to the *checkbox*, not to the underlying question.

### Revisit when

Two weeks of re-auth telemetry exist.

---

## D004: Tenant toggle is not enforced mid-session

**Date:** 2026-03-16
**Status:** Accepted
**Deciders:** JS

### Context

Disabling password auth doesn't kill existing sessions. Sessions are validated at creation, not
per request. Enforcing mid-session means a per-request tenant-config lookup, a change to shared
session infrastructure that isn't in this spec.

### Decision

Not enforced mid-session. Existing sessions run to natural expiry, maximum 8 hours.

### Consequences

An admin who disables password auth because of a suspected compromise has an 8-hour window where
it's still live. That's a real hole. It's documented, it's bounded, and the fix is session
revocation, which is a different project and should be one.

**This is a known gap, not a discovered one.** The difference is this entry.

### Revisit when

Session revocation is built, or an incident makes the 8-hour window unacceptable.

---

## D005: Password auth defaults to off for existing tenants

**Date:** 2026-03-17
**Status:** Accepted
**Deciders:** JS, K. Maiden

### Context

LOGIN-04 needed a default for tenants that already exist. Defaulting on adds an authentication
method to every account without anyone asking for it.

### Decision

Off. Admins opt in.

### Consequences

Slower adoption, and support will field "why can't I see this" tickets. Both acceptable. The
alternative was silently changing the security posture of every existing customer, which is the
kind of thing that reads badly in a post-incident write-up.

---

## D006: Whether a disabled tenant's admins may still reset a password

**Date:** 2026-04-09
**Status:** Proposed
**Deciders:** J. Studdart, K. Maiden

### Context

FR2 says a tenant can disable password auth. If it is disabled and the only admin has forgotten
their password, is reset a way back in, or is that exactly the hole the toggle closes? RESET-03 is
blocked on this.

### Options considered

| Option | Pros | Cons |
|---|---|---|
| A: reset honours the toggle absolutely | The toggle means what it says | Locked-out tenants need support |
| B: reset allowed for tenant admins only | Recoverable | A second path into a disabled method |

### Decision

Not yet taken.

### Consequences

RESET-03 blocked; AC5 unmapped until decided.
