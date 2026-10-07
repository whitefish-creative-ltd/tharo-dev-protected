# Definition of Done: Kestrel, auth work

**Version:** 3
**Agreed by:** JS, K. Maiden
**Date:** 2026-04-02 *(revised post-audit)*

> Version 3 exists because version 2 said "code is tested" and version 2 shipped DR3.

## The list

### Built
- [ ] Every AC on the feature spec has a test that asserts **the AC**, not the function.
- [ ] The traceability matrix has no unmapped ACs.
- [ ] Part 2 of the matrix is empty, or every row is logged in the drift register with a verdict.
- [ ] No files touched outside the task's stated scope.
- [ ] **The Not Building register has been read against the diff.** *(Added v3: DR1, DR2.)*

### Proven
- [ ] Tests assert behaviour, not implementation.
- [ ] Every scenario in spec §8 has a test.
- [ ] **Every prohibition in spec §9 has a negative test, including on the exception path.**
      *(Added v3: DR3. This is the line that would have caught it.)*
- [ ] Each test has been seen to fail. Break the behaviour, watch it go red, put it back.
- [ ] No test's name is doing work its assertions don't. *(Added v3.)*

### Recorded
- [ ] Decision log updated for anything non-obvious.
- [ ] Spec updated via CR if reality argued back. Not quietly.
- [ ] Not Building register unchanged, or changes logged as overturns with a CR reference.

### Operable
- [ ] Deployable by someone who didn't write it.
- [ ] Rollback tested, not assumed.
- [ ] When it breaks at 3am there's enough in the logs to know why, **and nothing in them there
      shouldn't be.** *(Added v3. Yes, this one is pointed.)*

### Reviewed
- [ ] Read by someone who didn't write it, including anything an agent wrote.
- [ ] The traceability matrix was produced by someone who didn't build it.

## Banned words

Any line containing these gets rewritten or deleted:

**Done, working, tested, clean, refactored, robust, secure, performant, documented, handled,
covered, sorted.**

## What v2 said, and what it cost

| v2 line | Why it failed | v3 line |
|---|---|---|
| Code is tested | `test_no_password_in_logs` existed and passed. It tested two of three paths. "Tested" was true and meant nothing | Every AC has a test that asserts the AC, and has been seen to fail |
| Errors are handled | The exception handler existed. It logged the password | Every prohibition in §9 has a negative test, including on the exception path |
| Scope respected | Nobody could say what that meant, so nobody checked | The Not Building register has been read against the diff |
| It's secure | Unfalsifiable. Ticked on every ticket, including the one with DR3 in it | *(Deleted. Replaced by the specific lines above)* |

Four lines. Every one was ticked on the pull request containing the plaintext password
disclosure. Every tick was honest: the developer believed each statement, and by the loose
meaning of the words, each was arguably true.

That's the failure mode. Not dishonesty. **Words that can be ticked in good faith while being
wrong.** A definition of done made of those isn't a gate, it's a ritual, and rituals don't stop
anything.

## Pruning note

v2 had 19 lines. v3 has 16, despite adding 5. Four were deleted for being unfalsifiable and
three for being things nobody had ever failed. A list nobody can complete gets ignored wholesale
and then you have no list at all.
