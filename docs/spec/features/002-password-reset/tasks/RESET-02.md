# RESET-02 — Link consumption and session invalidation

- **Objective:** Opening a valid link and submitting a new password replaces it, ends every session, and burns the link.
- **Implements:** FR3
- **Satisfies:** AC3, AC4
- **Depends on:** RESET-01
- **Effort:** M
- **Risk:** High — reuse and expiry must be tested on the exception path too.
- **Needs a human:** No
- **Out of scope for this task:** the toggle (RESET-03), the UI (RESET-04).
- **Status:** In progress · **Assignee:** R. Osei

**Done when:**
- [ ] AC3 and AC4 tested, including reuse after success and use after expiry.
- [ ] Session invalidation tested against a real Redis, not a mock.
