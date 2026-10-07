# RESET-01 — Reset request endpoint

- **Objective:** POST /auth/reset issues a single-use link for a known email and behaves identically for an unknown one.
- **Implements:** FR3
- **Satisfies:** AC1, AC2
- **Depends on:** LOGIN-01
- **Effort:** S
- **Risk:** Medium — the timing equivalence is the trap, as in LOGIN-02.
- **Needs a human:** No
- **Out of scope for this task:** the link-consumption endpoint (RESET-02), the toggle (RESET-03), the UI (RESET-04).
- **Status:** Done · **Assignee:** K. Maiden

**Done when:**
- [ ] AC1 and AC2 tested, timing test at 1,000 iterations.
- [ ] No token or email body in logs at any level.
