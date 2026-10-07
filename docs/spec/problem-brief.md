# Problem Brief: First-party login for Kestrel

**Status:** Agreed
**Date:** 2026-03-02
**Author:** J. Studdart

## 1. The problem

People who need to use Kestrel can't get into it. Access depends entirely on their employer's
identity provider being configured correctly, and for a large share of our customers it isn't,
so the people who bought the product can't reach it, and neither can anyone they invite.

## 2. Who has it

Two groups.

**Admins at customers without a working IdP**, roughly 40% of accounts, mostly the sub-50-seat
end. They bought Kestrel, they can't log in, and they have no IT department to fix it.

**Anyone they invite externally**, contractors, auditors, agency staff. Currently impossible.
Their identity isn't in the customer's directory, so there's nothing to federate. Support
handles about 15 of these a week by creating accounts manually, which is exactly as bad as it
sounds.

## 3. Evidence

- 47 support tickets in Q1 tagged `access`. 31 are "can't log in", none of which are bugs.
- Onboarding drop-off: 38% of new accounts never see a second session. Sales believes most are
  stuck at the IdP step; nobody has instrumented it, so this is inference.
- Two churned accounts in Q1 cited setup friction in their exit call. Small sample, consistent
  story.
- Support currently spends about a day a week on manual account creation.

## 4. Cost of the problem

Support time is the measurable part: ~1 day/week, call it £12k/year fully loaded.

The onboarding drop-off is the real number and we don't have it. If a third of the 38% are IdP
casualties, that's ~£180k ARR at current ACV, but that's an estimate resting on an inference,
and it should be treated as a reason to instrument rather than a reason to build.

## 5. Why now

Two things changed. The Q1 churn calls put it in front of the board. And the enterprise deal in
the pipeline needs external auditor access, which we currently cannot offer at all.

## 6. What happens if we do nothing

Support cost continues and grows with the account base. The enterprise deal is at risk on a
requirement we can't meet. The drop-off continues invisibly, which is the part that worries me
most, we'd never see it get worse.

Not catastrophic. Corrosive.

## 7. Constraints known up front

- **Immovable:** UK GDPR. We're the data controller for authentication data.
- **Immovable:** existing SSO must keep working, unchanged. Enterprise customers depend on it
  and will not tolerate a migration.
- **Expensive to move:** Postgres, existing session infrastructure, current tenant model.
- **Deadline:** enterprise deal needs a demonstrable answer by end of Q2.
- **Assumed:** that we must build this rather than buy an identity provider. **Not confirmed.**
  See open questions, this assumption is doing a lot of work and nobody has tested it.

## 8. What success looks like

- An admin at a customer with no IdP can reach a working account without contacting support.
- An external collaborator can be invited and get in.
- Support tickets tagged `access` drop below 5/quarter.
- SSO customers notice nothing at all.

## 9. Open questions

| Question | Why it matters | Owner | Needed by |
|---|---|---|---|
| Build or buy? Auth0/WorkOS vs. first-party | Changes everything below this line. Currently assumed, not decided | JS | 2026-03-09 |
| Is the 38% drop-off actually the IdP step? | The main cost justification rests on an inference | Product | 2026-03-09 |
| Does the enterprise deal need auditor SSO, or is email/password acceptable? | Might not need this at all for that deal | Sales | 2026-03-06 |

> **Note:** the last question is the cheap one and nobody had asked it. If the answer is "SSO",
> the deadline pressure evaporates and this becomes a Q3 problem. Ask before building.
