# LOGIN-03 — Rate limiting

- **Objective:** five failures locks an account for fifteen minutes.
- **Implements:** FR5
- **Satisfies:** AC6
- **Depends on:** LOGIN-02
- **Effort:** S
- **Risk:** Medium. The lock must not become an enumeration oracle. A locked account and a
  non-existent one must be indistinguishable.
- **Needs a human:** No
- **Out of scope for this task:** IP-based limiting. Per-account only, per FR5.
- **Status:** Done · **Assignee:** K. Maiden

**Done when:**
- [x] AC6 tested including the correct-password-while-locked case.
- [x] Lock state failure is fail-closed. If Redis can't be read, the attempt fails.
