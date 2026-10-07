# The Constitution: Kestrel

**Status:** Living document
**Owner:** J. Studdart
**Last updated:** 2026-03-14

> The standing rules for all Kestrel work. Every agent session points at this file. Every new
> engineer reads it on day one. It is not `CLAUDE.md`; that file exists and contains one line:
> "Follow constitution.md." Same for anyone using Cursor or anything else. One source of truth.

## How to use this file

- Agents: `CLAUDE.md` / `.cursorrules` both contain only "Follow constitution.md." Canonical copy
  is here, in the repo root.
- Humans: this is onboarding. Read it before your first PR.
- Review: every PR is checked against this and against the relevant spec.

---

## 1. Non-negotiables

- Never commit secrets, keys, or credentials. We use environment injection via the platform vault;
  nothing sensitive is ever in the repo, a branch, or a log.
- Never touch `billing/` or `auth/session/` without sign-off from JS or KM. These are the two
  places a mistake is expensive and hard to reverse.
- Never weaken a security control to make a test pass. If a test is failing because of a control,
  the test is wrong or the design is, and either way it's a conversation, not a bypass.
- Never add a dependency without flagging it first (see §4).

## 2. When you hit a decision that isn't specified

Do not decide it silently. Flag it, get it decided, log it in `decisions/`, then continue.

This has bitten us. See decision D004: the tenant toggle's mid-session behaviour was left
unspecified, someone made a reasonable assumption, and it became a known security gap that we now
carry deliberately instead of by accident. The difference between "known gap" and "surprise" is
whether someone flagged it here instead of guessing.

## 3. Coding standards

- **Languages:** Python 3.12 (backend), TypeScript 5.x (frontend).
- **Formatter / linter:** `ruff` (config in `pyproject.toml`), `biome` (frontend). Both
  authoritative. CI rejects unformatted code; don't relitigate style in review.
- **Naming:** modules and functions `snake_case`; the one rule tooling doesn't catch, a boolean
  is named for its true state (`is_locked`, not `lock_status`).
- **Structure:** feature-first. `auth/`, `billing/`, `tenants/`, not `models/`, `views/`,
  `services/`. Things that change together live together.
- **Error handling:** raise typed exceptions from the domain layer; the API layer maps them to
  responses. Never leak an internal exception to a client. Never leak a stack trace to a log at
  info level or above.
- **Logging:** structured JSON via the shared logger. Levels: error for things that page someone,
  warn for things to review, info for audit-relevant events. **Never log: passwords, tokens,
  reset links, full request bodies on the auth paths, any PII beyond a tenant and user ID.** This
  rule has a body count, see the drift register, DR3.
- **Comments:** explain why, never what. A comment restating the code is deleted in review.

## 4. Dependencies

- Flag new dependencies in the PR description before adding: what it's for, what it replaces, why
  not the stdlib.
- Discouraged: anything that reimplements something the framework already does; anything with a
  single maintainer and no releases in a year; anything that pulls a large transitive tree for a
  small feature.
- The reminder we keep learning: every dependency is capability in and risk in. This applies
  doubly to agent skills, see §8.

## 5. Testing

- Behaviour, not implementation. A test that breaks on a legitimate refactor is wrong.
- Every acceptance criteria has a test that asserts it.
- Every "must never happen" has a negative test, **including on the exception path.** DR3 is why
  this is in bold.
- Every test is watched failing once before it's trusted.
- Coverage floor is 80%, and coverage is a floor, not a verdict. 94% coverage shipped a plaintext
  password to the logs. The number is not the proof.

## 6. Git and change hygiene

- **Branching:** trunk-based. Short-lived feature branches, merged behind flags.
- **Commits:** conventional commits. The body says why.
- **PRs must include:** a link to the spec, the criteria IDs the PR satisfies, and an explicit
  line confirming the diff was read against the not-building register.
- **The scope check:** the not-building register is in the PR template as a checkbox. Added after
  DR1 and DR2: two features nobody asked for got through four reviews because everyone was
  reviewing for correctness and nobody was reviewing for scope.

## 7. Security posture

- Auth: sessions server-side, Redis-backed, 8-hour absolute expiry. SSO via the existing SAML
  library; do not modify it as a side effect of anything.
- Sensitive data: passwords (Argon2id only), tokens, reset links, PII. Handling rules in §3's
  logging line. Tenant isolation is enforced by row-level security; never write a query that
  could cross a tenant boundary.
- Secrets: platform vault, injected at runtime. Never in code, config, or logs.
- We are data controller for auth data under UK GDPR. When in doubt about a data-handling
  question, it's a conversation with JS, not a judgement call in a PR.

## 8. Working with agents

- Canonical rules are here. Tool configs point here and contain nothing else.
- An unrequested addition is drift, however helpful. The scope check in §6 exists to catch it.
- Skills are audited before use with the Skill Audit Checklist. Nothing unreadable in five minutes
  gets run against this repo. We had a near-miss with a "spec formatter" skill that wanted to read
  the environment, see the audit example.
- No agent verifies its own output. The audit is a separate pass, ideally a separate agent with no
  build context.

## 9. What this file is not

Not the spec, not system documentation, not a restatement of ruff's rules, not a home for
feature-specific rules. Standing rules only. Currently two pages. If it reaches three, something
in here belongs in a spec instead.
