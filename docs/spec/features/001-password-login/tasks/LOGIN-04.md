# LOGIN-04 — Tenant toggle

- **Objective:** an admin can turn password auth on or off for their tenant.
- **Implements:** FR2
- **Satisfies:** AC4
- **Depends on:** LOGIN-02
- **Effort:** S
- **Risk:** Medium. The disabled path must be checked on every route into a session, including
  password reset. A toggle that's honoured by login but not by reset is not a toggle.
- **Needs a human:** **Yes.** Default value for existing tenants. Turning it on by default
  changes the security posture of every account without asking. *(Resolved: default off. D005.)*
- **Status:** Done · **Assignee:** J. Studdart

**Done when:**
- [x] AC4 tested.
- [x] Reset flow also honours the toggle.
