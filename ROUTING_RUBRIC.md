# Routing Rubric — v1

This is the operator's judgment written down so a program can apply it the same way every time, and so a reviewer can see exactly why a request landed where it did. When the engine and the operator disagree, the operator is right and the disagreement is logged.

## 1. Lanes (exactly one per request)

| Lane | Meaning | What happens next |
|---|---|---|
| **Hold** | The request contains sensitive or privileged content that should not be processed further by software (personal health details, legal exhibits, credentials, financial account data, anything a reasonable person would call confidential *content*, not just the word) | Stop. No extraction beyond requester and subject. Human decides. |
| **Spam / invalid** | Unsolicited pitch, automated noise, empty or unreadable body, or no way to reply | Archive. No reply. |
| **Duplicate** | Same requester, same request, within 14 days — a resend, a "did you get this?", or a forward of the original | Link to the original. Reply only if the original is still unanswered. |
| **Out of scope** | A real request that the business does not do (see §4), or a request from an individual for personal rather than business work | Polite decline draft, with a pointer elsewhere if one is obvious. |
| **Needs info** | A plausible request missing one or more required fields (§3), or whose outcome cannot be stated in one sentence | Question draft asking for exactly the missing fields, nothing more. |
| **Qualified** | A business request within scope, with all required fields present, fit score ≥ 60 | One-screen brief plus a reply draft proposing the next step. |

## 2. Rule order

Rules are checked top to bottom. The first rule that fires sets the lane. This order is deliberate: safety first, then noise, then duplication, then scope, then completeness.

1. **Hold** if sensitive content is present in the body or attachments.
2. **Spam / invalid** if the body is empty or unreadable, there is no reply address, or the message is a pitch *to* the business rather than a request *of* it.
3. **Duplicate** if the same sender (or same company + same subject) already has a request in the last 14 days that describes the same work. A *different* request from the same sender is **not** a duplicate.
4. **Out of scope** if the ask is outside §4 or is personal rather than business.
5. **Needs info** if any required field (§3) is missing or the outcome is unclear.
6. **Qualified** otherwise, provided fit score ≥ 60. If fit < 60 and nothing above fired, route to **Needs info** and say what would raise it.

## 3. Required fields for Qualified

| Field | Why it is required |
|---|---|
| A reply address that works | Otherwise nothing can happen |
| A requester name or role | To address the reply and judge authority |
| An outcome stated in one sentence | "A comparison of three supplier bids" is an outcome; "help with AI" is not |
| Inputs the requester can provide | Files, data, access, or "none needed" — the request must say which |

**Optional but captured:** company, deadline, request type, domain, budget signal. Missing optional fields lower the fit score; they do not block Qualified.

## 4. In scope (v1 definition, generic)

Delegated knowledge work with a defined deliverable: an analysis, outline, comparison, review, release notes, proposal, launch plan, cost model, or a recurring version of one of these. Three request types:

- **Workflow** — focused piece of work, hours to a day
- **Project** — several workflows, one delivery, days to weeks
- **Service** — recurring or hosted work, an ongoing relationship

**Out of scope examples:** building software products from scratch, staffing or recruiting, personal (non-business) matters, anything requiring licensed professional advice delivered to an individual, physical goods.

## 5. Flags (added on top of the lane; never change it)

| Flag | Fires when |
|---|---|
| **urgent** | Deadline inside 3 business days, or the words make urgency explicit |
| **verify_company** | Free-mail sender (gmail, yahoo, etc.) or no company given, on an otherwise business request. A free-mail address is **not** a reason to downgrade the lane. |
| **language** | Request is not in English; records the language so the draft matches it |

Flags are not evaluated on **Hold** requests, because processing stops at the lane. In test fixtures, `freemail.example.net` stands in for consumer mail providers (gmail, yahoo, and the like), so no real address can appear in the test set.

## 6. Fit score (0–100), with reasons

The score explains *how good* a Qualified request is; it is not the gate for most lanes. Each component records a one-line reason.

| Component | Points | Guidance |
|---|---|---|
| Outcome clarity | 0–25 | 25 = one sentence, concrete deliverable · 10 = direction but no deliverable · 0 = unclear |
| Fit to offerings (§4) | 0–25 | 25 = squarely a workflow/project/service · 0 = outside scope |
| Domain match | 0–15 | 15 = clearly one of the six domains · 5 = general knowledge work |
| Deadline realism | 0–10 | 10 = stated and feasible · 5 = none stated · 0 = stated and infeasible |
| Requester identifiability | 0–15 | 15 = work email + company · 8 = free mail + company · 3 = free mail, no company (still valid) |
| Completeness of inputs | 0–10 | 10 = inputs named or "none needed" · 0 = unknown |

**Threshold:** Qualified at ≥ 60. Below 60, Needs info with the lowest-scoring components named in the draft.

## 7. Draft-reply rules

- One template per lane. Variables are filled **only** from fields present in the request. An absent field becomes a question, never a guess.
- About half of each draft is fixed text written by a human; the engine fills the rest.
- Every draft begins `DRAFT —` and is never sent by the engine.
- Hold-lane requests get no draft; they get a note to the operator.

## 8. What counts as a pass in testing

A test case **passes** when the engine's lane equals the expected lane. Flags are scored separately. A pass with a wrong flag is logged as a flag miss, not a routing miss. Fit scores are checked for direction (above/below 60), not for the exact number.

## 9. Change log

- **v1 (2026-10-09):** six lanes, three flags, six-part fit score, rule order fixed. Labels in `test_requests_v1.csv` are frozen against this version.
