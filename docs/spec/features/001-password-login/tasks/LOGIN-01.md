# LOGIN-01 — Credential storage

- **Objective:** a password can be stored against a user and verified, with nothing else changed.
- **Implements:** FR1 (partial), NFR1
- **Satisfies:** none yet *(no user-facing AC; verified by unit tests on the module)*
- **Depends on:** nothing
- **Effort:** S
- **Risk:** Low
- **Needs a human:** No
- **Out of scope for this task:** the login endpoint, sessions, rate limiting, the UI.
- **Status:** Done · **Assignee:** K. Maiden

**Done when:**
- [x] Argon2id at NFR1 parameters, verified against known vectors.
- [x] `password_hash` nullable, where null means "no password credential", not "not set yet".
- [x] Verify function is constant-time and hashes a dummy on the not-found path.
- [x] Decision log updated if parameters deviate from NFR1. *(They didn't.)*
