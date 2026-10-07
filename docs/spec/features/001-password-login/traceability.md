# AC Traceability Matrix: Password login

**Spec version:** Kestrel first-party login v1.0
**Date:** 2026-04-01
**Audited by:** K. Maiden *(did not build it)*

## Part 1: Specified vs. proven

| AC | Requirement | Test | Automated | Result | Notes |
|---|---|---|---|---|---|
| AC1 | FR1 | `test_login_success` | Yes | Pass | |
| AC2 | FR1 | `test_login_wrong_password` | Yes | Pass | |
| AC3 | FR1 | `test_login_timing_equivalence` | Yes | Pass | 1,000 iterations, means within 12ms |
| AC4 | FR2 | `test_login_tenant_disabled` | Yes | Pass | |
| AC5 | FR1 | `test_login_sso_user_no_password` | Yes | Pass | |
| AC6 | FR5 | `test_rate_limit_lockout` | Yes | Pass | |
| AC7 | FR1 | `test_session_tenant_scoped` | Yes | Pass | |
| AC8 | FR1 | `test_login_redis_down` | Yes | Pass | Real container stopped, not mocked |
| AC9 | NFR4 | `test_no_password_in_logs` | Yes | **FAIL** | **Test does not assert the AC. See below** |

### AC9: the finding

The test exists, is named correctly, has passed for eleven days, and does not test the acceptance
criteria.

AC9 requires that no password reaches the logs on **any** login attempt, *including one that
throws*. The test covers the success path and the wrong-password path. Both are handled. Neither
throws. The exception path, the only one where a serialised request body could plausibly reach
a logger, is not exercised.

It isn't hypothetical: `logger.debug(request.json())` is sitting in the exception handler right
now. See DR3.

**A green tick next to AC9 for eleven days.** The test name was the only thing anyone read. If
this matrix had been produced when LOGIN-02 was marked done rather than at release, the finding
would have cost an hour instead of a rollback.

**Verdict:** AC9 is unmapped. The test is theatre and is being rewritten to force the exception.

## Part 2: Built but not specified

| What | Where | Traces to | Verdict |
|---|---|---|---|
| Remember-me checkbox, 30-day cookie | `LoginForm.tsx`, `session_service.py` | Nothing. Contradicts NB6 | **Revert**, DR1 |
| Password strength meter (zxcvbn) | `InviteForm.tsx` | Nothing. Contradicts NB8 | **Revert**, DR2 |
| `logger.debug(request.json())` | `auth/views.py:87` | Nothing. **Violates AC9 and spec §9** | **Revert immediately**, DR3 |
| IP-based rate limiting | `auth/limiter.py` | Nothing. Out of scope per LOGIN-03 | **Ratify**, DR4. Correct, and the spec was wrong |

Four rows. Three shouldn't be there, one should have been in the spec.

Note that three of these four passed code review. Reviewers read diffs for correctness (does
this code do what it appears to intend?) and all four of these did. Nobody was reading the diff
against the Not Building register, because until this audit there was nowhere that comparison
happened. Reviewing for correctness and reviewing for scope are different jobs and only one of
them was being done.

## Part 3: Specified but absent

| AC | Requirement | Status | Why |
|---|---|---|---|
| (none) | FR6 (SSO + password coexist) | Not built | Deferred. Should, not Must. Logged for next sprint |
| (none) | FR7 (last-login display) | Not built | Deferred by CR001. Could |

Both deferred deliberately, both traceable to a decision. No "missed" rows this time, which
tells us the task-to-done gate is working even though the done-to-verified gate isn't.

## Part 4: Summary

- ACs specified: **9**
- ACs with a passing test that actually asserts them: **8**
- ACs unmapped: **1** *(AC9, had a passing test the whole time)*
- Shipped behaviour with no AC: **4** *(3 revert, 1 ratify)*
- **Verdict: FINDINGS.** Release blocked pending DR3 revert and AC9 test rewrite.

## What this audit cost and what it bought

Ninety minutes.

It found a plaintext password disclosure into the log aggregator that a full green test suite,
94% line coverage, and four code reviews had all walked past. It also found two features nobody
asked for and one the spec should have had.

The test suite was never going to find DR3, because the test suite was written by the same
process that wrote the bug and it agreed with itself. That's the case for an auditor who didn't
build it, not paranoia about agents specifically, but the ordinary observation that nothing
checks itself well. It applies identically to a human who's been staring at the same code for a
fortnight.
