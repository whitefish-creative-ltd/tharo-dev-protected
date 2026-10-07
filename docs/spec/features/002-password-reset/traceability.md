# AC Traceability Matrix: Password reset

**Spec version:** Kestrel first-party login v1.0
**Date:** 2026-04-09
**Audited by:** K. Maiden *(did not build it)*

## Part 1: Specified vs. proven

| AC | Requirement | Test | Automated | Result | Notes |
|---|---|---|---|---|---|
| AC1 | FR3 | `test_reset_request_known_email` | Yes | Pass | |
| AC2 | FR3 | `test_reset_request_unknown_email_timing` | Yes | Pass | 1,000 iterations, means within 9ms |
| AC3 | FR3 | none | none | **UNMAPPED** | RESET-02 in progress |
| AC4 | FR3 | none | none | **UNMAPPED** | RESET-02 in progress |
| AC5 | FR2 | none | none | **UNMAPPED** | RESET-03 blocked on D006 |

## Part 2: Built but not specified

| What | Where | Traces to | Verdict |
|---|---|---|---|
| | | | |

## Part 3: Specified but absent

| AC | Requirement | Status | Why |
|---|---|---|---|
| AC5 | FR2 | Not built | Blocked — waiting on the toggle decision (D006) |

## Part 4: Summary

- ACs specified: 5
- ACs with a passing test: 2
- ACs unmapped: 3
- Shipped behaviour with no AC: 0
- **Verdict:** Findings
