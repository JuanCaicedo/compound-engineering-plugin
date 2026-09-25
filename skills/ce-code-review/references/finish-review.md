### Stage 5: Merge findings

Convert multiple reviewer compact JSON returns into one deduplicated, confidence-gated finding set. Use `scripts/findings-mechanics.py` from this skill's directory for schema/value validation, exact-fingerprint deduplication, conservative route merging, quote/confidence gates, deterministic sorting, and stable numbering. These are mechanics, not model judgment.

Write the compact reviewer returns as a JSON array, then run the command below exactly. Do not inspect the helper source or run its `--help`; its contract is this reference and its JSON output.

Input is one JSON array of reviewer-return objects. Output is one object with `findings`, `pre_existing_findings`, `suppressed_findings`, `suppressed_by_confidence`, `malformed_findings`, and `malformed_returns`; use those fields directly and do not inspect implementation to infer additional behavior.

```bash
SKILL_DIR="<absolute path of the directory containing the SKILL.md you just read>";
PY="$(for c in python3 python py; do command -v "$c" >/dev/null 2>&1 && "$c" -c '' >/dev/null 2>&1 && { echo "$c"; break; }; done)"; [ -n "$PY" ] || { echo "no working Python 3 interpreter on PATH" >&2; exit 1; };
"$PY" "$SKILL_DIR/scripts/findings-mechanics.py" < "$RUN_DIR/raw-returns.json" > "$RUN_DIR/mechanical-findings.json"
```

Before the first helper run, load every available per-reviewer artifact and build a source-detail map keyed by reviewer plus the helper fingerprint: normalized `file`, string `line`, and whitespace-normalized lowercase `title`. The map owns each source finding's `why_it_matters` and `evidence`; compact returns are merge inputs, not final report objects.

Inspect the helper's `findings`, `pre_existing_findings`, and `suppressed_findings` for semantic duplicates that use different wording or nearby anchors, for the direct-dependency exception, and for settlement conflicts below. Merge only when candidates describe the same defect and fix path. If semantic reconciliation, direct-dependency reclassification, or settlement stamping changed the set, serialize every reconciled candidate from all three partitions as one valid synthetic reviewer return and run the helper again to restore deterministic gates, partitions, sort order, and numbering. Never ask the helper to decide semantic equivalence or settlement conflicts. When reconciling a semantic duplicate, carry its original source-map keys alongside the candidate in working memory so detail hydration does not depend on the rewritten title. The helper's deterministic `suppressed_findings` partition is not primary review output; after settlement reconciliation, inspect it for the soft-bucket route below, then discard the remainder while preserving `suppressed_by_confidence` counts.

Then apply only the judgment the helper cannot own:

1. **Semantic reconciliation.** Merge differently worded findings only when they describe the same defect and fix path. Keep disagreements visible. Union the mechanics-produced `reviewers` and `independent_reviewers` lists from the merged candidates; never add an identity to `independent_reviewers` merely because it appears in `reviewers`. A pre-existing gap stays primary only when the new change directly depends on it for correctness; mark that reconciled candidate `pre_existing: false` before the final helper pass. Nearby cleanup remains pre-existing.
2. **Settled decisions.** Inspect both surviving `findings` and `suppressed_findings`. If a finding merely prefers an alternative to a `session-settled:` KTD, stamp `settled_conflict`, route it advisory/human, and include it in the synthetic rerun so the helper preserves it in the primary report. Never apply it. Do not demote a real defect or evidence that the settled approach cannot work. Honor inferred-plan settlements only when the match is unambiguous.
3. **Restore mechanics.** After semantic reconciliation, direct-dependency reclassification, or settlement stamping, rerun the helper with every reconciled candidate, including unchanged primary and pre-existing candidates plus stamped and unstamped suppressed candidates. It enforces the quote-the-line gate, discrete confidence anchors, exact dedup, independent-agreement promotion, conservative routing, pre-existing partition, confidence gate, deterministic sort, and stable `#` numbering. `fast-pass` never promotes confidence. With cross-model peer review disabled in this fork, no reviewer carries different-family independence, so independent-agreement promotion is available only between distinct in-process personas.
4. **Soft-bucket demotion before validation.** After the final helper pass, inspect both surviving `findings` and the remaining `suppressed_findings`. Keep every `settled_conflict`-stamped finding primary as required by Stage 5 step 2.
   - A current P0/P1 remains primary unless it is a semantic duplicate, validated false, preference-only settled conflict, or genuinely pre-existing. Never move it to `residual_risks` or `testing_gaps` merely because only one reviewer found it. Findings with different failure modes or fix paths are not duplicates even when they touch the same lines (for example, awaiting a ledger append does not solve a later publish-failure retry that appends twice).
   - For testing-only absence-of-coverage findings, keep at most one umbrella primary finding per changed subsystem when the lack of tests is itself material; move narrower case-by-case coverage findings to `testing_gaps` regardless of their persona severity.
   - Move a single-reviewer P2/P3 advisory from `testing` to `testing_gaps`, and from `maintainability`, `reliability`, or an adversarial reviewer to `residual_risks`, unless it quotes an explicit violated contract or proves a current user-facing defect.
   - Do not widen a repository contract with an assumed deployment topology or process lifetime. A claim that requires unproven restarts, multiple instances, or infrastructure behavior is a residual risk unless the changed code or repository evidence establishes that operating condition and its violated guarantee.
   - Suppressed candidates routed here remain absent from primary `findings`; discard all other `suppressed_findings` after this step. Record the mode-aware demotion count. Only the remaining primary set enters Stage 5b.

**Detail hydration gate (before validation and rendering).** Hydrate every retained primary and pre-existing finding from its source-detail map entries. For exact merges, union and deduplicate the contributing `evidence` arrays and keep the most specific source `why_it_matters`; for semantic merges, use the carried original source-map keys. `actionable_findings` later reuses these hydrated objects rather than rebuilding a compact subset. Every retained finding must have a non-empty `why_it_matters` string and non-empty `evidence` array. If an artifact is unavailable or malformed, re-read the cited changed line and use `first_evidence` when present to reconstruct only directly verified detail. Never invent impact from the title alone. If the required fields still cannot be established, drop the candidate as malformed before Stage 5b and record it in the coverage record; never emit a partial finding in markdown, `mode:agent`, or `review.json`.

5. **Partition work.** The actionable queue is `gated_auto` or `manual` plus owner `downstream-resolver`; advisory and human/release-owned findings are report-only. Reuse the helper's stable `#` everywhere.
   A concrete current P0/P1 with a specific code or test response belongs to `downstream-resolver` even when the subsystem is financially or operationally sensitive; the downstream caller still reviews, applies, and verifies it. Use owner `human` only when the next step genuinely requires a product/design decision, unavailable authority, external coordination, or release action. Reviewer caution and `requires_verification:true` do not by themselves remove a fixable defect from the caller's actionable queue.
   Before rendering, normalize every concrete P0/P1 to `downstream-resolver` unless its report entry names the specific product/design choice, unavailable authority, external dependency, or release decision that blocks implementation. A broad redesign, several related edits, or sensitive code is not such a blocker. The Actionable Findings section contains only the resulting `downstream-resolver` queue; human/release-owned items appear as decision gates outside it.
6. **Build thematic triage groups.** Group related findings so the reader can triage themes instead of items. This is distinct from deduplication: groups never merge findings or change severity, confidence, route, owner, or stable `#`. Groups span the full primary set.
   - **`grouping:off`:** skip this step.
   - **`grouping:auto` (default):** build groups when findings span distinct concerns — the trigger is distinct concerns, not item count (mirroring how plan Requirements group by capability). Skip only when all findings are genuinely about the same thing; prefer no groups over decorative single-item groups.
   - **`grouping:always`:** always build groups; use single-finding groups only when no meaningful multi-finding grouping exists.
   - **Grouping signals:** shared root cause, affected subsystem, user-facing failure mode, overlapping fix path, dependency ordering, or repeated symptoms of one design choice.
   - **Group shape:** short title, the included stable finding `#`s, one-line context, preferred resolution, and why — when one fix path resolves several findings, name it and say which finding to handle first.
   - **Ordering:** order groups by the highest-severity finding they contain, then by lowest stable `#`. A finding appears in at most one group; leave genuinely unrelated findings ungrouped.
7. **Collect coverage and advisory assets.** Keep helper drop/suppression counts, union residual risks and testing gaps, and preserve any selected learnings, agent-native, and deployment-verification outputs. Schema drift from `data-migration` is already in the merged finding set.

### Stage 5b: Validation pass (optional quality gate)

Independent verification remains required for findings that lack corroboration. With cross-model peer review disabled in this fork, no finding arrives with cross-family corroboration, so the cross-family shortcut never applies here.

1. Skip a validator only when the finding has `first_evidence` and both an ordinary reviewer plus an `adversarial-<provider>` reviewer whose artifact records `independence_verified:true`. Same-model corroboration never licenses this shortcut. Record the skip count in the coverage record (run artifacts, not the report).
2. Select every remaining P0/P1 and every remaining actionable finding, except preference-grade `settled_conflict` items. Single-source P0s are always selected. P2/P3 advisory items should already have moved to the soft buckets; do not validate them merely to keep them primary.
3. Put the selected set into **one** deterministic validator batch, ordered by severity and stable `#`, using `references/validator-batch-template.md`. Eight findings is the normal cap. When more than eight P0/P1 survive, expand that same batch past the normal cap to include every surviving P0/P1 rather than silently omitting a blocker; never split the work into another batch. The validator returns one independent verdict per finding. Cost, elapsed time, confidence, or a finding appearing mechanically obvious never licenses an additional skip; when the required validator cannot run, keep P0/P1 as validation-degraded and say so.
4. Run the validator batch foreground with background execution off. A foreground Agent call is the wait. Never use shell no-ops (`echo waiting`, `noop`, `yield turn`, `end turn`, `true`, or sleeps), scheduled wakeups, status/list calls, or narrated "still waiting" turns.
5. A valid `validated:false` drops that finding and records the reason. On malformed output or validator infrastructure failure, drop affected P2/P3 findings; keep affected P0/P1 as validation-degraded. Prune triage groups after drops and record the batch, per-finding verdicts, failures, and degraded blockers in the coverage record.

### Stage 5c: Act on findings (explicit local apply only)

**Skip unless local apply was explicitly authorized.** A bare `ce-code-review` invocation is report-only and does not apply findings. Authorization exists only when `apply:local` was passed or the invoking user prompt explicitly asked this review to apply/fix its findings. Do not infer authority from `autofix_class`, a clean tree, an actionable finding, or the fact that another workflow may apply later. `mode:agent` does not apply fixes and conflicts with `apply:local`; the pipeline caller owns any later mutation.

`apply:local` is authority, not an output mode: presentation remains markdown and reviewer selection is unchanged.

**Act policy (bias to act).** Default to applying every finding that is a clear improvement and a reversible edit, regardless of severity. The work is a tracked, visible diff that can be reverted — so leaving a clean fix unapplied "to be safe" is the failure mode, not the safe choice. Decide by judgment, not a safety checklist:

- **Apply** clear improvements — the common case (test hardening, dead-code removal, a localized fix with a concrete `suggested_fix`).
- **Push back** — do not apply — when the reviewer is wrong; keep the finding and state the disagreement with reasoning.
- **Skip with judgment** taste calls and conflicting suggestions, but surface what was skipped and why. Never silently drop.

Severity, confidence, and cross-reviewer agreement tell you what to do first and what to flag loudly — they do not gate the decision. There is no deny-list: downside is controlled after the fact (revert + visible diff + the commit checkpoint), not by a precondition.

One exception: `settled_conflict`-stamped preference findings (Stage 5 step 2) stay report-only even when local apply is authorized — the bias-to-act rule does not apply to them. The user already chose against that alternative; reversing it is not this review's improvement to make.

**Scope invariant.** Apply only when the working tree *is* what was reviewed — `local-aligned` or standalone. In `pr-remote` / `branch-remote` the working tree is not the reviewed head; do not apply — report instead.

**Verify, then keep.** After applying, run the affected tests and lint (targeted by default; broaden when fixes span files). If they fail, revert that fix and report it as a finding instead — an unverified fix is not finished. Never leave the tree red.

**Review the autofix diff before finishing.** Before committing or reporting applied fixes, diff only the changes introduced during Stage 5c against the pre-apply checkpoint. Run one self-review pass over that diff:
- If the same helper, policy, or guard was added to multiple parallel surfaces, extract it or explain in the Applied section why duplication is intentional.
- If an exported/shared function now accepts a broader input shape, update the nearby docs, types, or tests that define the contract so future callers understand it.
- If a reviewer item is pure information (no defect, no code contract change, no test gap), classify it as advisory/non-actionable in the coverage record or residual risks; do not patch it or describe it as a missed defect.
If this self-review changes files, rerun the affected tests or lint for those follow-up edits before committing or reporting; the earlier validation only covers the original autofix diff.

**Commit when the pre-review tree was clean.** Before applying, note whether the working tree already had uncommitted changes (`git status --porcelain`). The permanence gate is the **push**, not the commit — a local commit is private and reversible (`git reset --soft HEAD~1`).

- **Clean before the review:** after applying and verifying, commit the fixes as one isolated, review-labeled fix commit — `fix(review): <summary>`, or the repo's nearest convention if `review` isn't an allowed scope. Labeled and reversible, returning the tree to a known state.
- **Dirty before the review:** apply but do **not** commit — the fixes interleave with the user's in-flight work and ride along with the commit they were already going to make. The Applied section lists what changed.
- **Never push, open a PR, or file tickets** — that's the outward-facing step the user owns.

**Surface green-but-unverifiable edits.** When an applied fix touches auth/authz, a public or cross-service contract/schema, or concurrency/ordering, a passing test does not prove safety — flag it prominently in the Applied section so the diff reviewer's attention goes there.

**Re-partition triage groups after apply.** Triage groups describe the *remaining* work. After Stage 5c, prune applied findings out of `triage_groups` before Stage 6 rendering — a group must never tell the user to handle a finding that was already applied. When an applied fix resolved part of a theme, note that in the group's context line instead of keeping the applied `#` in the group. Re-apply the Stage 5 step 6 grouping rule (drop sub-two-finding groups under `grouping:auto`).

### Stage 6: Synthesize and present

Assemble the final report. **Default:** human-readable markdown, shaped as an **action list**. **`mode:agent`:** skip markdown and emit JSON (see ### JSON output format) — the structured fields are how a downstream agent consumes the review, and the JSON keeps the `verdict`, `coverage`, `learnings`, `requirements_completeness`, `residual_risks`, and `testing_gaps` fields that the markdown no longer renders as sections.

**Report completion gate:** do not finish until every surviving finding appears as an action item carrying its stable `#`, `file:line`, severity, reviewer(s), confidence, and route; every `downstream-resolver` finding is one of those items (never silently replaced by a count); the report **ends on the highest-priority action item**, with nothing informational after it; and the markdown contains **no Verdict block, no Coverage section, no Learnings section, no blockquote, and no horizontal rule**. In `mode:agent` the exact JSON fields replace this gate.

**Before writing, load `references/review-output-template.md` and mirror its section skeleton** — that file is the canonical skeleton for *which sections appear and in what order*; its example shows one good rendering, not the only permitted layout. The direction below is the always-loaded fallback so it survives a long session even if the template was not reloaded.

**Presentation direction — optimize for the reader's next action (goal + considerations, not a fixed layout).** The report is *acted on*: by a human deciding what to fix, or by a downstream agent applying fixes. Shape it so that action is fast and well-founded.

Write human-readable findings in an ASD-STE100 Simplified Technical English (STE)-inspired style. Use short, direct sentences. Keep one consequence, action, or supporting idea per sentence, and use one consistent term for each concept. Preserve exact code identifiers, error text, and domain terms. Shorten sentences, not content: preserve coverage, evidence, technical depth, and every distinct consequence or required action.

- **The report is read bottom-up, so it is written in reverse order of importance.** When output ends the terminal viewport sits on the last line, so the last thing printed must be the thing to do first. Context and already-settled information — scope, intent, reviewers, artifact path, Notes, pre-existing issues, applied fixes, triage groups — goes at the **top**, where it is scrolled past. The action items go **last**, buckets running `Optional` -> `Worth fixing` -> `Fix before merge`, so the blocking work is on screen without scrolling. Nothing informational follows the last item: no recap, no artifact path, no closing remark.
- **Every item is read cold, so every item is self-sufficient.** The reader scrolls to an item and acts on it there. Title, `file:line`, severity, reviewer(s), confidence, and route travel with the item; the consequence and the response are in its body. Never defer meaning to a block above it ("see the Notes", "per Coverage").
- **Per item, make four things unambiguous:** *what & where* — an imperative title plus `file:line`, not the mechanism; *why it matters* — what breaks or who is hit, in observable behavior, never a restatement of the code; *what response it needs* — this varies by finding type: a bug states its fix, a **design call** presents the options and the tradeoff without forcing one answer and is tagged `decision`, a coverage gap names the test and precedent to mirror, an already-applied item gives what changed and how it was verified; *how sure* — the confidence anchor, and whether it was corroborated (cross-reviewer agreement is the strongest signal available here — say so).
- **No summary sections at all.** The verdict, the coverage inventory, and the learnings roundup are gone from the markdown. Their content either becomes an item, becomes one `Notes` line at the top per the template's Notes rule, or stays in the run artifacts. Do not re-list the items anywhere — the buckets are the handoff.
- **Fold the advisory prompt outputs into the items.** A `learnings-researcher` result that identifies a defect becomes its own item; one that applies to an existing item becomes a `Notes` line referencing that `#`; one with no action attached is dropped. `agent-native-reviewer` gaps become items at their own severity. `deployment-verification-agent` output becomes the `### Deploy checks` bucket — blocking pre-deploy checks, the verification queries that matter, and rollback caveats, each phrased as an action. Schema drift stays a `data-migration` finding, never a separate section.
- **Group by the unit of work or decision, not just severity.** Buckets order urgency; they do not tell the actor what *kind* of action an item needs. Surface the split: **decisions a human must make** (design calls, ambiguous semantics — tagged `decision`) vs **mechanical work that can just be done** (tests, dedup, concrete fixes) vs **informational** (advisory items) — an agent clears the mechanical work and must stop at the decisions. Group items sharing a root cause or one fix (the Triage Groups) and name the order/dependency ("decide #3 once -> unblocks #2; do #3 first"); the unit of work is often a group, not an item.
- **Detail is earned by enabling the next action, not by demonstrating thoroughness.** Cover *every* finding — completeness is non-negotiable — but say each in the least that lets the consumer act. **Do not paste file contents or re-print the diff**; it is already in the repo/PR — cite `file:line` and spend words only on what the diff cannot show (why it breaks, the fix, the repro). This governs *expression, never coverage*: never drop a finding or its why/fix to be shorter, and match weight to weight (a nit is one line; a P1 design call earns room).
- **Let the shape serve the content; stay consistent within a section.** Items are prose blocks with a tag group; Applied and Triage Groups are compact tables. Pick what reads clearest for that content and keep one shape inside a section.

**Hard constraints (non-negotiable; everything above is judgment):**
- **No blockquotes and no horizontal rules.** Terminal renderers shade an entire `>` block, so the most important text renders least readably, and a `---` rule now separates nothing.
- **ASCII-safe only — no box-drawing or per-item horizontal-rule separators (`────`, `———`), no Unicode arrows or middot (`·`); use `->`.** These break across terminals and violate repo convention.
- **Inline code wraps identifiers, paths, and short snippets only** — never a whole clause or sentence, which renders as a solid highlighted block.
- **Stable `#` numbering from Stage 5** — never re-derive per bucket; reuse the same `#` everywhere a finding appears. A multi-file applied fix is one row with one `#`, never duplicated.
- **If you use a markdown table, escape literal `|` in cells as `\|`** so a pipe inside a title/regex/cache-key example doesn't split the row.
- **Reverse urgency order is a constraint, not a preference.** Buckets run `Optional` -> `Worth fixing` -> `Fix before merge`; within a bucket, items run from lowest confidence anchor to highest. The report's last line belongs to the highest-confidence blocker.
- **Every actionable finding is an item the reader can act on in place**, carrying severity, `file:line`, the imperative what, the consequence, and its route. There is no separate actionable recap; in default mode, when the actionable queue is empty, `Actionable findings: none.` is the report's last line.

Render the sections in this order — informational first, most urgent last:

1. **Header.** `## Code Review -- <N> action items, <M> blocking` (`M` = the `Fix before merge` bucket; drop the clause when 0), then scope, intent, mode, the reviewer team with per-conditional justifications, and `Artifacts:` with the resolved run-artifact path. The path lives here, not at the end — the end belongs to the blocking work.
2. **Notes.** At most five lines, per the template's Notes rule: a past learning tied to an item, a scope or trust caveat (untracked files excluded, lite roster, failed or malformed reviewer return, inferred intent, base override), a one-line plan statement, or an open question for the author. Unaddressed plan requirements are items in the buckets, not a checklist here.
3. **Not from this change.** `pre_existing: true` findings, one line each. They are never counted in the blocking total.
4. **Applied (explicit local apply only).** When Stage 5c applied fixes, list them in an Applied section (see review output template); each entry carries `#`, file, the fix, and reviewer (a multi-file fix is one row with one `#`), then a one-line validation outcome (e.g. "pin tests 4 -> 6; suite 94 pass, lint clean") and commit status (committed on a clean tree as `fix(review): …` or the repo's nearest convention, or left uncommitted for the user on a dirty one). Flag green-but-unverifiable edits (auth/contract/concurrency) prominently. Applied work is already done, so it sits above the remaining work. Omit this section when local apply was not authorized or nothing was applied. Applied findings appear here, not in the buckets.
5. **Triage Groups.** When finalized `triage_groups` exist (post-validation, post-apply — Stage 5b step 5 / Stage 5c), render a `### Triage Groups` section before the findings as a compact table (`| Group | Findings | Context | Preferred Resolution | Why |`) — a table fits this content well. The `Findings` cell lists the stable `#`s it covers; the resolution names the order/dependency. **Mark whether each group is an apply-queue or a decision-gate** (so an automated fixer applies the mechanical groups and stops at the design calls). Every referenced `#` must appear as an item below; groups supplement the findings, never replace them. Omit the section when `grouping:off` is active or no groups survived. In `mode:agent` this section is carried by the `triage_groups` JSON field instead.
6. **Deploy checks.** Only when the `deployment-verification-agent` prompt ran: its Go/No-Go items as actions. Omit otherwise.
7. **Action items, bucketed by ascending urgency.** `### Optional` (P3), then `### Worth fixing` (P2), then `### Fix before merge` (P0 + P1) as the **final section of the report**. Omit empty buckets. Within a bucket, order by confidence anchor **ascending**, so the last item printed is the highest-confidence blocker. Finding numbers come from the stable assignment in Stage 5 -- never re-derive them per bucket or triage group; `#1` stays the most urgent item, so the numbers count down the screen. A dependency between items is stated in the body ("decide #3 first"), never by position.

**Requirements completeness routing.** When Stage 2b found a plan, unaddressed requirements or implementation units become items, routed by `plan_source`: **`explicit`** (caller-provided or PR body) -> P1, `autofix_class: manual`, `owner: downstream-resolver`, so they enter the actionable queue; **`inferred`** (auto-discovered) -> P3, `autofix_class: advisory`, `owner: human`, report-only, because an inferred plan match is a hint, not a contract. When every requirement is addressed, that is the one-line plan statement in Notes. When no plan was found, say nothing about plans anywhere.

**Coverage is recorded, not rendered.** Write the full coverage record to the run artifacts (and to the `mode:agent` JSON `coverage` field): applied count, suppressed count by anchor, mode-aware demotion count, validator batch outcome per selected finding plus drop count and reasons, P0/P1 kept under degraded validation, the quote-the-line demotion count, residual risks, testing gaps, failed or timed-out reviewers, the Stage 3c lite roster when it ran, excluded untracked paths, scope mode, inferred-intent uncertainty, settlement-suppression state, removable surface, and the fact that the review ran entirely in-process with no cross-model corroboration. None of that is a report section. Only a coverage fact that changes whether the items can be trusted or whether a lens ran at all is surfaced, as one `Notes` line.

Do not include time estimates.

**Final check before delivering (default only).** Verify the hard constraints, not a layout: no blockquote and no horizontal rule anywhere; no box-drawing / per-item separators (`────`), no Unicode arrows or middot (`·`); no Verdict, Coverage, or Learnings section; stable `#`s consistent across sections; literal `|` escaped (`\|`) in any table cell; **the last line of the report belongs to the highest-priority action item** (or to `Actionable findings: none.`), with the artifact path and every informational section above the buckets; and **every item self-sufficient where it sits** — a reader landing on one item gets the imperative what, `file:line`, severity, confidence, route, and the consequence without scrolling anywhere else. Re-render anything that fails. Skip when `mode:agent` is active.

After the final artifact write returns, emit the final response immediately. The artifact write is the last tool call; never use `true`, `echo`, a placeholder transition, or any other tool call to create another turn before the final response.

### JSON output format (`mode:agent` only)

Emit **one raw JSON object** as the primary response — a single bare JSON value, **no markdown code fence**. A leading ```` ```json ```` fence makes the response start with backticks and breaks naive `JSON.parse` consumers, so never wrap it. Also write `review.json` under the resolved `<run-dir>` with the same payload.

`mode:agent` does not apply fixes — the caller does — so there is no `applied_fixes` field; the handoff is `actionable_findings`. Applied work surfaces only in explicitly authorized local-apply markdown runs (Stage 5c/6).

Minimum shape:

```json
{
  "status": "complete",
  "verdict": "Ready to merge | Ready with fixes | Not ready",
  "scope": {
    "base": "<merge-base sha, pr:NNN marker, or base: ref>",
    "branch": "<current branch name>",
    "head_sha": "<git rev-parse HEAD>",
    "pr_url": "<url or null>",
    "files_changed": 0
  },
  "intent": "<2-3 line summary>",
  "intent_confidence": "explicit | inferred | uncertain",
  "reviewers": ["correctness", "security"],
  "findings": [],
  "actionable_findings": [],
  "triage_groups": [],
  "pre_existing_findings": [],
  "requirements_completeness": null,
  "learnings": [],
  "agent_native_gaps": [],
  "deployment_notes": [],
  "residual_risks": [],
  "testing_gaps": [],
  "coverage": {},
  "artifact_path": "<resolved-run-dir>",
  "run_id": "<run-id>"
}
```

Each object in `findings` uses the merged finding fields: `#`, `title`, `severity`, `file`, `line`, `confidence`, `autofix_class`, `owner`, `requires_verification`, `pre_existing`, `suggested_fix`, `first_evidence`, `why_it_matters`, `evidence`, `reviewers`, `independent_reviewers`. The helper derives `independent_reviewers`; synthesis may preserve or union that list but must not infer it from `reviewers`.

Findings stamped by the Stage 5 step 2 settlement-conflict rule additionally carry an optional `settled_conflict` field naming the conflicting `session-settled:`-labeled KTD (its identifier or name). The field is absent on findings with no settlement conflict; consumers that do not recognize it ignore it.

`actionable_findings` lists the `gated_auto` / `manual` + `downstream-resolver` subset with the same fields plus stable `#`.

Each object in `triage_groups` carries `{ "title", "findings": [<stable #s>], "context", "preferred_resolution", "why" }` — the finalized groups from Stage 5 step 6 after Stage 5b step 5 pruning. Every referenced `#` must exist in `findings` (the full set) — **not** necessarily in `actionable_findings`. Groups are a triage **lens over all findings, not an apply queue**: a group (and its `preferred_resolution` ordering) can reference advisory or `human`/`release`-owned findings that the caller must not apply. So a caller batching related fixes by theme must first intersect each group's `findings` with `actionable_findings` and act only on that subset — the apply handoff stays `actionable_findings`, never `triage_groups`. Empty array when `grouping:off` is active or no groups were built.

On failure before review completes, set `"status": "failed"` and `"reason": "<one sentence>"`. When all reviewers fail, use `"status": "degraded"` with a reason. When a PR skip rule fires (closed/merged/trivial), use `"status": "skipped"` with the skip reason. Do not emit markdown when `mode:agent` is active.

## Quality Gates

Before delivering the review, verify:

1. **Every finding is actionable.** Re-read each finding. If it says "consider", "might want to", or "could be improved" without a concrete fix, rewrite it with a specific action. Vague findings waste engineering time.
2. **No false positives from skimming.** For each finding, verify the surrounding code was actually read. Check that the "bug" isn't handled elsewhere in the same function, that the "unused import" isn't used in a type annotation, that the "missing null check" isn't guarded by the caller.
3. **Severity is calibrated.** A style nit is never P0. A SQL injection is never P3. Re-check every severity assignment.
4. **Line numbers are accurate.** Verify each cited line number against the file content. A finding pointing to the wrong line is worse than no finding.
5. **Protected artifacts are respected.** Discard any finding that recommends deleting or gitignoring a CE pipeline artifact, per the Protected Artifacts rule in SKILL.md: any file under a `plans/`, `solutions/`, or legacy `brainstorms/` directory whose immediate parent is the artifact root (a directory named `docs`, or the configured `docs_root` when resolved). Categories nest (`solutions/<category>/`); a `references/personas/` skill asset, parented by `references`, is not a protected artifact.
6. **Findings don't duplicate linter output.** Don't flag things the project's linter/formatter would catch (missing semicolons, wrong indentation). Focus on semantic issues.
