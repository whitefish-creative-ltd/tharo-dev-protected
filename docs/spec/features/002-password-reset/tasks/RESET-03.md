# RESET-03 — Tenant toggle honoured by reset

- **Objective:** A tenant with password auth disabled gets the generic response and no email.
- **Implements:** FR2
- **Satisfies:** AC5
- **Depends on:** RESET-01, LOGIN-04
- **Effort:** S
- **Risk:** Medium — a toggle honoured by login but not reset is not a toggle.
- **Needs a human:** **Yes.** Whether a disabled tenant's admins should still be able to reset in order to re-enable — open as D006.
- **Out of scope for this task:** everything but the toggle check.
- **Status:** In progress

**Done when:**
- [ ] AC5 tested.
- [ ] D006 recorded before merge.
