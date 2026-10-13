# Design decisions

Why the engine is shaped the way it is. Each entry is the decision, the reasoning, and what would change it.

## 1. Inbound triage first, not outbound

Inbound requests are demand that already exists. The most expensive leak in a small B2B business is a qualified request that waits two days for a vague reply, and the gap is rarely measured. Fixing that is the smallest system with a measurable outcome: one input, one decision per request, one number (time to a specific reply) that the business already cares about. Outbound systems depend on the same data model and the same discipline, so they come second and reuse this one.

## 2. Lanes, and why these six

A lane is a decision that leads to a different next action. Lanes exist because each one triggers a different reply template and a different owner. Six is the smallest set in which every request has exactly one home:

- **Qualified** — reply with a proposed next step.
- **Needs info** — ask for exactly what is missing. Kept separate from Out of scope because the next action is a question, not a decline.
- **Out of scope** — decline politely, point elsewhere. A real request for the wrong business.
- **Duplicate** — link to the original. Resends are common; left unhandled they inflate volume and produce two conflicting replies.
- **Spam / invalid** — archive. No reply exists for a pitch or an empty message.
- **Hold** — stop processing. Software should not extract, summarize, or draft from sensitive or privileged content; a person decides what to do with it.

What would change this: a client whose requests regularly need a seventh action (for example, "forward to a partner firm"). Add the lane only when a real next action requires it.

## 3. One lane, many flags

Urgency, a free-mail sender, or a non-English request each change *how* a request is handled, not *where* it goes. If those were lanes, the category count would multiply (urgent-qualified, urgent-needs-info, …) and routing accuracy would become unmeasurable. So the lane is the single decision, and flags ride on top. A free-mail address, in particular, is a reason to verify, never a reason to downgrade: plenty of real owners of small businesses write from personal addresses.

## 4. A fixed rule order

Rules fire top to bottom and the first match wins: Hold, then Spam/invalid, then Duplicate, then Out of scope, then Needs info, then Qualified. The order encodes priorities. Sensitive content is never archived as spam because a spam rule happened to match first. A resend of a declined request is still a duplicate, so the original decision is reused instead of re-argued. Fixed order also makes the engine deterministic: the same request routes the same way every time, which is what makes a test set meaningful.

## 5. Fit score and lane are separate

The lane answers "what happens next." The fit score answers "how good is this request," with a one-line reason per component. Keeping them separate means each can be measured and tuned on its own. Teams that merge scoring and routing into one number tend to lose the ability to explain why conversion changed. The score gates only the last step (Qualified needs ≥ 60); every other lane is decided by rules a person can read.

## 6. Four required fields, no more

A reply is possible with a working address, a name or role, an outcome stated in one sentence, and the inputs the requester can provide. Everything else — company, deadline, budget — is captured when present and lowers the score when absent, but never blocks a request. Each additional required field lowers the share of requests that get through, and the cost of a question is lower than the cost of a lost request.

## 7. Unknown stays unknown

When the engine cannot find a field, it records *unknown*. It does not infer a company from an email domain, a title from a signature line, or an email from a name. A guessed value is acted on as if it were true; the cost shows up later as a misdirected reply or a damaged sender reputation. A gap is visible and cheap to fill with one question.

## 8. Templates with variables, not free-written replies

Each lane has one template written by a person; the engine fills the variables from fields the request supplied and turns any missing field into a question. Roughly half of every draft is fixed text. This keeps voice and promises consistent, makes replies comparable across requests, and makes a change to the template a change to every future reply rather than a hope.

## 9. A human approves every reply

Replies are external communication. A wrong send costs a relationship, not a retry. So the engine prepares; the operator decides; sending stays manual. The approval step is also the feedback loop: every correction is logged with the original, and the pattern of corrections is what improves the rubric in the next version.

## 10. An event log and one SQL question, rather than a dashboard

The log is the product. Every stage, flag, correction, and approval is timestamped in a small database, and every metric is derived from it. SQLite in v1 keeps setup at zero; the SQL is written to run unchanged on Postgres later. One precisely answered operating question — volume, median time to approved brief, and first-pass approval by request type and lane — is worth more than a page of charts nobody can reproduce.

## 11. Synthetic data first

Edge cases can be designed deliberately instead of waiting months for a real inbox to produce them: the resend versus the genuinely new request from the same sender, the free-mail owner with a real ask, the request that says "confidential" but contains nothing sensitive next to the one that pastes patient records. Building on a synthetic set separates "does the logic work" from "does it work on this client's mix," and the labeled set becomes the regression test every later change must pass. Live results come from shadow mode, where the engine runs beside the human and nothing changes for requesters until the comparison holds up.

## 12. Every number carries a label

Observed, Tested, Modeled, or Planned KPI. The label says how a number was measured, so a reader never has to guess whether a figure came from live use, a bounded test, or a projection. This is the same discipline a client would expect in a diligence call, applied from the first commit.
