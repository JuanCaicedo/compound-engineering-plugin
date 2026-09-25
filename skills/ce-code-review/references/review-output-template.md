# Code Review Output Template

The report is an **action list**, ordered for a terminal reader. Every item is something the reader will either fix (their own change) or raise as review feedback (someone else's). Anything the reader cannot act on does not belong in the report. This file is the **canonical skeleton** for *which sections appear and in what order* — copy the section structure; the example below shows one good rendering, not the only permitted layout.

**The report is read bottom-up, so it is written in reverse order of importance.** When output ends, the terminal viewport sits on the last line — so the last thing printed is the thing to do first. Context and already-settled information (scope, reviewers, artifacts, applied fixes, pre-existing issues, caveats) goes at the **top**, where it is scrolled past. The action items go **last**, least urgent first, so the blocking work is on screen without scrolling.

**Hard constraints (non-negotiable; the rest is judgment):**

- **Every item stands on its own where it is read.** Title, `file:line`, severity, reviewer(s), confidence, and route travel with the item. A reader who sees only one item knows what breaks, how sure the review is, and who acts next — never "see the verdict below" or "per the Coverage section".
- **Nothing informational after the items.** No Verdict block, no Coverage section, no Learnings section, no recap of the same items, no artifact path, no closing remark. Their content either becomes an action item, moves to the header or `Notes` at the top, or is dropped from the report and left in the run artifacts. The last line of the report is part of an action item.
- **Reverse urgency order.** Buckets run `Optional` -> `Worth fixing` -> `Fix before merge`, and within a bucket items run from lowest confidence anchor to highest. The final item printed is the highest-confidence blocker.
- **No blockquotes.** Terminal markdown renderers shade every word of a `>` block, so the text most worth reading renders least readably. Emit every section as plain text.
- **No horizontal rules** (`---`) and no per-item separators.
- ASCII-safe throughout: no box-drawing (`────`), no Unicode arrows or middot (`·`); use `->`.
- Every item is one action, phrased as an imperative, with a `file:line` citation. Never paste file contents or re-print the diff.
- Inline code wraps identifiers, paths, and short snippets — never a whole clause or sentence. A long backticked span renders as a solid highlighted block in a terminal.
- **Stable `#` numbering from Stage 5** — `#1` is the *most* urgent item, so the numbers are the fix order and match the `mode:agent` JSON. Because the report prints in reverse urgency, the numbers count **down** the screen and `#1` is the last item shown. Never re-derive them per bucket; reuse the same `#` wherever a finding reappears (Applied, Triage Groups, cross-references between items). A multi-file applied fix is one row with one `#`.
- **If you use a markdown table, escape literal pipe characters in cells.** Any `|` inside a title, description, code snippet, regex pattern, or delimited-string example (e.g. cache key examples like `userName + "\|" + groups`) must be written as `\|` so column boundaries are determined only by unescaped pipes. Unescaped pipes split the cell across columns and corrupt the rest of the row.

## Sections

Top to bottom, ending on the most urgent item:

| Section | Contents |
|---------|----------|
| Header | Action-item count and blocking count, scope, intent, mode, reviewer roster, and the run-artifact path |
| `### Notes` | At most 5 lines, only per the Notes rule below |
| `### Not from this change` | `pre_existing: true` findings, one line each |
| `### Applied` | Explicit local apply only — fixes already made, with validation and commit status |
| `### Triage Groups` | Only when finalized groups exist — related items and the order to resolve them |
| `### Deploy checks` | Only when the deployment-verification prompt ran — pre-deploy, verification, and rollback actions |
| `### Optional` | P3 items |
| `### Worth fixing` | P2 items |
| `### Fix before merge` | P0 and P1 items — **last section in the report** |

Omit any section with nothing in it. Within a bucket, order items by confidence anchor **ascending**, so the best-evidenced item is closest to the reader's cursor. A dependency between items is stated in the body ("decide #2 first"), never expressed by position — position carries urgency only.

When the actionable queue is empty there is no bucket to print; say `Actionable findings: none.` as the report's last line instead.

## Example

```markdown
## Code Review -- 5 action items, 3 blocking

**Scope:** merge-base with the review base branch -> working tree (14 files, 342 lines)
**Intent:** Add order export endpoint with CSV and JSON format support
**Mode:** markdown + explicit local apply
**Reviewers:** correctness, testing, maintainability, security, api-contract
- security -- new public endpoint accepts a user-provided format parameter
- api-contract -- new /api/orders/export route with response schema
**Artifacts:** `/tmp/compound-engineering-501/ce-code-review/20260821-142233/`

### Notes

- The team has hit #4 before: `<root>/solutions/export-pagination.md` documents the same unbounded-export pattern.
- The `maintainability` reviewer returned malformed JSON and was dropped, so structural quality went unreviewed on this diff.
- All 6 requirements in `<root>/plans/order-export.md` are addressed by the diff.

### Not from this change

- `orders_controller.rb:12` -- broad `rescue` masks a failed permission check (P2, correctness)

### Applied

| # | File | Fix | Reviewer |
|---|------|-----|----------|
| 6 | `export_helper_test.rb:40` | Added the missing test for the empty-format branch | testing |
| 7 | `orders_controller.rb:88` (+test) | Tightened export file perms `0644 -> 0600` (security-posture — verify in diff) | security |

Validation: export tests 11 -> 13; suite 214 pass, lint clean.
Committed: `fix(review): cover empty-format branch + tighten export perms` (working tree was clean before review).

### Triage Groups

| Group | Findings | Context | Preferred Resolution | Why |
|-------|----------|---------|----------------------|-----|
| Export result-set scaling | #2, #3 | Both stem from loading the full order set in one pass; decision-gate | Decide the pagination contract (#3), then stream behind it (#2) | One cursor/page decision resolves the memory bound and the API shape together |

### Deploy checks

- Capture baseline row counts before enabling the export backfill.
- Verify `SELECT COUNT(*) FROM exports WHERE status IS NULL;` stays at `0` after rollout.
- Keep the old export path available until the backfill is validated (rollback path).

### Optional

**5. Give the export an agent-invocable entry point** -- `orders_controller.rb:30` (P3, agent-native, confidence 50, advisory -> human)
The export is reachable only through the web form, so agent and CLI users cannot trigger it. Expose it through the existing `bin/export` command surface if agent parity matters for this endpoint.

### Worth fixing

**4. Return a 4xx on CSV serialization failure** -- `export_service.rb:45` (P2, correctness, confidence 75, gated_auto -> downstream-resolver)
A serialization error escapes as a 500 with no partial-write cleanup, so a caller cannot tell a bad `format` parameter from a server fault. Rescue `CSV::MalformedCSVError` around the row writer and return 422 with the offending column.

### Fix before merge

**3. Decide the export pagination contract** -- `export_service.rb:91` (P1, api-contract + performance, confidence 75, manual -> human, decision)
The endpoint returns every row in one response, so response size grows without bound and the shape cannot change after GA without breaking clients. Two options: a cursor (`next_cursor` in the payload, stable under concurrent writes, no total count) or page/limit (simpler for clients, drifts under writes). Decide this before #2 lands, since it fixes the streaming boundary.

**2. Stream the export result set instead of materializing it** -- `export_service.rb:87` (P1, performance, confidence 100, gated_auto -> downstream-resolver)
`Order.where(...).to_a` loads every row into memory, so a large account OOMs the worker. Replace it with `find_each`, or paginate behind the contract decided in #3.

**1. Scope the export lookup to the current account** -- `orders_controller.rb:42` (P0, security + correctness, confidence 100, gated_auto -> downstream-resolver)
`find(params[:id])` on the export path has no ownership scope, so any authenticated user can export another account's orders. Add the `current_account.orders` scope, matching the guard in `shipments_controller.rb:38`. Both reviewers flagged this independently.
```

## Header

`## Code Review -- <N> action items, <M> blocking`, where `M` counts the `Fix before merge` bucket. Drop the `, <M> blocking` clause when `M` is 0. With no items at all, use `## Code Review -- no action items` plus one plain line naming what ran: the reviewer roster found nothing actionable, which is not the same as a clean bill of health.

Follow the count line with `Scope`, `Intent`, `Mode` (`markdown report-only`, `markdown + explicit local apply`), the reviewer roster with a one-line justification per conditional reviewer, and `Artifacts:` with the resolved run-artifact path when artifacts were written. The path belongs here, at the top — it is reference information, and putting it after the items would take the bottom line away from the blocking work.

## Item body

One to three sentences, in this order:

1. **What breaks** -- the finding's `why_it_matters`, in terms of observable behavior: a wrong result, an error path, a contract mismatch, an undetected regression. Never a restatement of the code.
2. **The response it needs** -- this varies by finding type. A bug states its fix as a concrete instruction, naming the symbol, guard, or test to add. A **design call** presents the options and the tradeoff without forcing one answer, and carries `decision` in its tag group. A coverage gap names the test and the precedent to mirror. "Consider refactoring" is not an action; if no action can be named, the item does not ship.
3. **How sure**, when it is not obvious from the tag group — cross-reviewer agreement (the strongest corroboration available in an in-process review), or the boundary a `residual_risks` entry leaves open (for example, callsite completeness established by grep only). One clause inside the body, never a separate section.

Keep the tag group `(severity, reviewer(s), confidence NN, autofix_class -> owner[, decision])` on the title line; move it to its own line only when the title line wraps. When an item depends on another, name it in the body ("decide #3 first") — the print order carries urgency, not sequencing, so a prerequisite may appear below the item it unblocks. Confidence is an integer anchor (`50`, `75`, `100`) — never a float. Multiple reviewers listed means independent agreement; say so in the body when it changes how sure the reader should be.

Match weight to weight: a nit is one line, a P1 design call earns room. Cover **every** surviving finding — economy governs expression, never coverage.

## Triage Groups

Render `### Triage Groups` above the item buckets, and only when finalized groups exist (`grouping:auto` found distinct concerns, or `grouping:always`). The `Findings` cell lists the stable `#`s it covers, and the `Context` cell says whether the group is an **apply-queue** (mechanical work an agent can clear) or a **decision-gate** (a human must decide first). Every referenced `#` must appear as an item below. Groups are a triage lens over the items -- they never replace the buckets, merge findings, or renumber them. Omit the section when `grouping:off` is active or no groups survived Stage 5b/5c pruning.

## Notes rule

`### Notes` sits near the top, under the header, because it is context rather than work. It exists only for facts that change whether the items below apply or whether the list can be trusted. At most five lines, one line each:

- A **past learning** that applies to a specific item (reference it by `#`) or that names a documented pattern the diff repeats. A learning that produces no action is dropped, not reported.
- A **scope or trust caveat** — the local tree is behind the remote branch, the diff base was overridden, untracked files were excluded, the lite roster ran, a reviewer failed or returned malformed JSON so its lens went unreviewed, or intent was inferred rather than stated.
- A **plan line** when a plan was discovered: whether every requirement is addressed. Unaddressed requirements are action items in the buckets above, not a checklist here.
- An **open question for the author** that no code change resolves.

Do not use Notes for suppressed-finding counts, validator batch outcomes, demotion counts, removable-surface totals, residual-risk inventories, the cross-model status, or a restatement of the items. Those belong in the run artifacts (and the `mode:agent` JSON `coverage` field), not in the report.

## Anti-patterns

Do NOT produce output like this. The following is wrong:

```markdown
Findings

Sev: P1
File: foo.go:42
Issue: Some problem description
Reviewer(s): adversarial
Confidence: 75
Route: advisory -> human
────────────────────────────────────────
Sev: P2
File: bar.go:99
Issue: Another problem
```

This fails because of the **box-drawing `────` separators between items**, no stable finding numbers, no bucket `###` headers, no `## Code Review` title, and no imperative action or consequence anywhere — a reader learns that something is wrong but not what breaks or what to do. `Field:`-prefixed lines are not themselves banned; carrying no stable numbers, no action, and box-drawn separators is the problem.

Also wrong, even with correct structure: a trailing `### Coverage` / `### Learnings & Past Solutions` / blockquoted `> **Verdict:**` block, or the buckets printed most-urgent-first so the P0 sits far above the last line. Both make the reader scroll back to reconstruct the gist. Fold informational content into the header or Notes at the top, and end the report on the highest-priority item.

## Agent mode (JSON)

When `mode:agent` is active, **do not** emit the markdown report above. Emit **one parseable JSON object** as the primary response and write the same payload to `review.json` under the resolved `<run-dir>`.

The contract is defined in SKILL.md under **`### JSON output format (`mode:agent` only)`**. Minimum fields: `status`, `verdict`, `scope`, `intent`, `reviewers`, `findings`, `actionable_findings`, `artifact_path`, `run_id`.

Key differences from the human-facing markdown format:

- **The JSON keeps the fields the markdown drops** — `verdict`, `coverage`, `learnings`, `requirements_completeness`, `residual_risks`, `testing_gaps`. Programmatic callers consume them structurally; the human report does not carry them as sections.
- **No buckets or prose items** — findings are JSON arrays with merged fields (`#`, `title`, `severity`, `file`, `line`, `confidence`, `autofix_class`, `owner`, `suggested_fix`, `why_it_matters`, `evidence`, `reviewers`, etc.).
- **`actionable_findings`** — subset for caller apply workflows (`gated_auto` / `manual` with `downstream-resolver`).
- **`triage_groups`** — the markdown Triage Groups section serialized as `{title, findings: [<stable #s>], context, preferred_resolution, why}` objects, so callers can batch related fixes by theme. Groups span the full finding set — a triage lens, not an apply queue — so a caller must intersect each group's `findings` with `actionable_findings` before applying; the apply handoff stays `actionable_findings`. Empty when `grouping:off` or no groups.
- **No `applied_fixes` and no Applied section** — `mode:agent` does not apply fixes; the caller does. Applied work surfaces only in explicitly authorized local-apply markdown (Stage 5c/6). The handoff is `actionable_findings`.
- **Failure/degraded paths** — `{"status":"failed","reason":"..."}` or `"status":"degraded"` with reason; never mix markdown into the JSON response.
- **Stable `#`** — same numbering as Stage 5 synthesis, carried in JSON finding objects for downstream apply/residual tracking.
