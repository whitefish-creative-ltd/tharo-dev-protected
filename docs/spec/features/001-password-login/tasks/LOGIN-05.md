# LOGIN-05 — Session creation and failure modes

- **Objective:** sessions are created correctly and login fails closed when Redis is gone.
- **Implements:** FR1 (completion)
- **Satisfies:** AC7, AC8
- **Depends on:** LOGIN-02
- **Effort:** S
- **Risk:** Low
- **Needs a human:** No
- **Status:** Done · **Assignee:** K. Maiden

**Done when:**
- [x] AC8 tested with Redis actually stopped, not mocked. A mocked outage tests the mock.
