# Evaluation Rubric

The orchestrator applies this rubric to every item **before** dispatching any fixer. Judging centrally — with all threads from a single fetch in view — is what catches a systematically-wrong reviewer or bot across threads, and lets the author's design intent be weighed against the finding. An isolated subagent cannot see that.

Read the referenced file for each item before classifying it. Classifying from the comment text alone is how a confidently-wrong finding gets "fixed."

## Decision order

Work these in order; the first one that applies wins.

1. **Is this a question or discussion?** The reviewer is asking "why X?" or "have you considered Y?" rather than requesting a change.
   - Answerable confidently from the code and context → `replied`
   - Answer depends on product/business decisions you cannot determine → `needs-human`

2. **Does the concern actually hold?** Does the issue the reviewer describes exist in the code as written?
   - No → `not-addressing`, citing the evidence (the guard that already exists, the line that already handles it)

3. **Is it still relevant?** Has the code at this location changed since the review was posted?
   - No longer applies → `not-addressing`

4. **Would the fix make the code worse?** The suggestion is coherent but the change would introduce a regression, fight an established pattern, or trade a real property away for a cosmetic gain.
   - Yes → `declined`, citing the specific harm

5. **Does the change buy anything real?** Pure churn — a rename with no readability gain, a reshuffle with no behavioral or clarity difference.
   - No → `replied`

6. **Would fixing improve the code?**
   - Yes → approve for fix
   - Uncertain → **approve for fix.** Agent time is cheap; default to fixing.

## Escalate to `needs-human` when

- The change is architectural and affects other systems
- The decision is security-sensitive and you cannot bound the risk
- The business logic is genuinely ambiguous
- Reviewers conflict with each other
- The fix would change behavior the author appears to have chosen deliberately

This should be rare — most feedback has a clear right answer. `needs-human` never blocks: leave the thread open, post a natural reply carrying the condensed analysis, and report the structured `decision_context`.

## Default to fixing

Most review feedback — nitpicks included — is correct and worth fixing. Validation is a tripwire, not a gate: you read the code to make the fix anyway, so divert only on a concrete signal. Do not manufacture doubt or risk to avoid work.

Judge every item on its merits regardless of source (human or bot) or form (diff thread or plain MR comment).

## Non-convergence

When feedback is *not converging* — several nits sharing a single **root** (the approach itself is the problem, e.g. "your regex misses case X" repeated for X after X), or a bot re-posting fresh nits on every commit without end — raise **one** approach-level `needs-human` about the root decision and stop fixing the individual instances.

Hold the anti-cry-wolf line: this fires only on a *demonstrated* shared root or a *demonstrated* treadmill across passes. A normal batch of unrelated valid nits is just fixed, one pass, as usual.
