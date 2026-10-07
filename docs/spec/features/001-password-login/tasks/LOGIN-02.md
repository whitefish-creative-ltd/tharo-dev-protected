# LOGIN-02 — Login endpoint

- **Objective:** POST /auth/login exchanges correct credentials for a session.
- **Implements:** FR1
- **Satisfies:** AC1, AC2, AC3, AC5, AC7, AC9
- **Depends on:** LOGIN-01
- **Effort:** M
- **Risk:** **High.** AC3 (timing) and AC9 (logging) are the two most likely things to be
  quietly wrong here, and both are invisible in a passing test suite unless the test is written
  to look for them specifically.
- **Needs a human:** No
- **Out of scope for this task:** rate limiting (LOGIN-03), tenant toggle (LOGIN-04), the UI
  (LOGIN-06). **Do not add a "remember me" option. See NB6.**
- **Status:** Done · **Assignee:** J. Studdart

**Done when:**
- [x] Every listed AC has a test that asserts it.
- [x] AC3's timing test runs 1,000 iterations and asserts means within 50ms.
- [x] AC9 asserted by capturing logs at debug and grepping the fixture password.
- [x] No files touched outside `auth/`.
