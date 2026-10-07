# Change & Drift Register: Kestrel first-party login

**Status:** Living document, append only
**Owner:** J. Studdart

## Part 1: Change requests

### CR001: Allow admins to see failed login attempts per user

- **Raised by:** Support lead
- **Date:** 2026-03-24
- **Status:** Accepted

**What:** extend FR7 (last-login time) to include a count of failed attempts in the last 24
hours, visible on the tenant admin user list.

**Why now:** support are fielding "why is my colleague locked out" and can't answer without a
database query. Three occurrences in the first week of the beta.

**Affects:** FR7, new AC10, new task LOGIN-07.

**Cost:** S. One query, one column, one test.

**What it displaces:** LOGIN-07 pushes FR7's last-login display to the following sprint. FR7 was
a Could. Nothing else moves.

**Non-goals affected:** none.

**Decision:** accepted. It's small, it's inside the existing surface, and it removes a support
burden the feature itself created.
**Decided by:** JS
**Date:** 2026-03-25

---

### CR002: Add "remember me" to the login form

- **Raised by:** Product
- **Date:** 2026-03-30
- **Status:** **Rejected**

**What:** a checkbox extending session life to 30 days.

**Why now:** beta users are complaining about re-authenticating. *(This is real, see D003.)*

**Affects:** NB6, D003, the session infrastructure, AC7.

**Cost:** looks like S. Isn't. A 30-day session needs revocation, device listing, and a
different threat model. Realistically M plus a security review.

**What it displaces:** unclear, which is itself the answer.

**Non-goals affected:** NB6 directly.

**Decision:** rejected *as framed.* The complaint is legitimate and D003 predicted it, but the
answer to "8 hours is too short" is to examine the expiry with the telemetry we're now
collecting, not to bolt a 30-day cookie onto a decision nobody has revisited. A checkbox is not
an analysis.

Raised as a Q3 piece of work: *Session lifetime, examined.* Which is what D003 said would need
to happen.
**Decided by:** JS
**Date:** 2026-03-31

> This is what the register is for. The complaint was real, the proposed fix was a reflex, and
> without NB6 and D003 sitting there in writing, "just add remember me, every site has it" wins
> that conversation on a Friday afternoon.

---

## Part 2: Drift log

| # | What drifted | Spec said | Reality is | Found by | Ratify or revert | Date |
|---|---|---|---|---|---|---|
| DR1 | Remember-me checkbox in the login form, 30-day cookie | NB6, explicitly not building | Built anyway during LOGIN-06 | Audit (Template 08, Part 2) | **Revert** | 2026-04-01 |
| DR2 | Password strength meter (zxcvbn) on the invite form | NB8, explicitly not building | Built during LOGIN-06 | Audit | **Revert** | 2026-04-01 |
| DR3 | `logger.debug(request.json())` in the login exception handler | AC9, never log a password, at any level, including in a stack trace | Logs the full request body, password included, on any unhandled exception | Audit | **Revert. Immediately.** | 2026-04-01 |
| DR4 | IP-based rate limiting alongside per-account | FR5, per account. LOGIN-03 explicitly scoped IP-based out | Both implemented | Audit | **Ratify**, see below | 2026-04-01 |

### DR1 and DR2: the helpful additions

Both were added while building LOGIN-06, unprompted. Both are on the Not Building register with
reasons. Neither was malicious, or even unreasonable in isolation: every login form on the
internet has a remember-me box and a strength meter, so a literal, capable thing building a
login form produces one with a remember-me box and a strength meter. It pattern-matched, which
is what it does.

**What should have caught this, and why didn't it?**

The task (LOGIN-06) *did* name both exclusions explicitly, and they got built regardless. So the
instruction wasn't the gate; nothing checked the output against the instruction. The audit
caught it, which is the gate working, but the audit is the last one before release. That's late.

**Action:** the Not Building register is now in the PR template as a review checkbox. Cheap, and
it moves the catch forward by a week.

### DR3: the one that matters

A debug log line in an exception handler, serialising the request body. Passwords into the log
aggregator, in plaintext, on any unhandled exception in the login path.

Every test passed. Coverage was 94%. The AC9 test asserted no password appeared in the logs on
the *success* and *wrong-password* paths, both of which are handled, neither of which throws.
The exception path wasn't tested, because writing a test that forces an unhandled exception in
login felt contrived.

**What should have caught this, and why didn't it?**

The AC9 test was written to the letter of the AC and not to its intent. AC9 says "including one
that throws". The test author read it, wrote two of the three cases, and the reviewer read the
test name, `test_no_password_in_logs`, and believed it.

This is the whole argument for the traceability matrix in one row: **a test that passes but
doesn't assert the AC is worse than no test**, because it buys false confidence at full price.
There was a green tick next to AC9 for eleven days.

**Action:** reverted. AC9's test now forces an exception. `logger.debug` of any request object
is banned by lint rule in the auth package.

### DR4: the one worth keeping

IP-based rate limiting was added alongside the specced per-account limiting. It's out of scope
for LOGIN-03, and it's also correct: per-account limiting alone does nothing against a spray
attack across many accounts from one source.

**Ratified.** FR5 amended to cover both. AC11 added. The spec was wrong, and this time the
guess was better than the brief.

### The ratify-to-revert ratio

Three reverts, one ratify. That's a healthy number. If it were three ratifies and one revert,
the honest reading wouldn't be "our agent has great judgement", it would be that our specs are
too thin to disagree with, and we're writing them after the fact, which is documentation, which
means we've stopped doing this.
