# Quick Review Output Template

The report is an **action list**. Every item is something the reader will either fix (their own change) or raise as review feedback (someone else's). Anything the reader cannot act on does not belong in the report.

**Hard constraints (non-negotiable; the rest is judgment):**

- **No blockquotes.** Terminal markdown renderers shade every word of a `>` block, so the text most worth reading renders least readably. Emit every section as plain text.
- **No horizontal rules** (`---`) and no per-item separators.
- ASCII-safe throughout: use `->`, not Unicode arrows or middots.
- Every item is one action, phrased as an imperative, with a `file:line` citation. Never paste file contents or re-print the diff.
- Numbering runs `1..N` in display order, so the numbers are the fix order. Reuse a number wherever the item reappears.
- Inline code wraps identifiers, paths, and short snippets — never a whole clause or sentence. A long backticked span renders as a solid highlighted block in a terminal.
- No Coverage section, no Learnings section, no Verdict block. Their content either becomes an action item or is dropped.

## Buckets

| Bucket | Contents |
|--------|----------|
| `### Fix before merge` | P0 and P1 findings |
| `### Worth fixing` | P2 findings |
| `### Optional` | P3 findings |
| `### Not from this change` | `pre_existing: true` findings, one line each |
| `### Notes` | At most 3 lines, only per the Notes rule below |

Omit any bucket with nothing in it. Order items within a bucket by confidence anchor (descending). When one item must be fixed before another, put the prerequisite first and say so in its body.

## Example

```markdown
## Quick Review -- 4 action items, 2 blocking

**Scope:** merge-base with origin/main -> working tree (14 files, 342 lines)
**Intent:** Add order export endpoint with CSV and JSON format support

### Fix before merge

**1. Scope the account lookup to the current account** -- `orders_controller.rb:42` (P0, correctness, confidence 100)
`find(params[:account_id])` on the export path has no ownership scope, so any authenticated user can export another account's orders. Add the `current_account.orders` scope, matching the guard in `shipments_controller.rb:38`.

**2. Return a 4xx on CSV serialization failure** -- `export_service.rb:87` (P1, correctness, confidence 75)
A serialization error escapes as a 500 with no partial-write cleanup, so a caller cannot tell a bad `format` parameter from a server fault. Rescue `CSV::MalformedCSVError` around the row writer and return 422 with the offending column.

### Worth fixing

**3. Test the format-dispatch branch** -- `export_service.rb:91` (P2, quality, confidence 75)
Both new format paths are untested, so a regression in either ships undetected. Add two request specs asserting the `Content-Type` and first row for `csv` and `json`.

### Optional

**4. Inline the `ExportHelper` wrapper** -- `export_helper.rb:12` (P3, quality, confidence 50)
The class is a single pass-through to `ExportService#serialize` and adds a hop with no behavior. Call the service directly and delete the file.

### Not from this change

- `orders_controller.rb:12` -- broad `rescue` masks a failed permission check (P2, correctness)

### Notes

- The team has hit #3 before: `<root>/solutions/testing/format-dispatch-coverage.md` documents the same untested-branch pattern.
- No security or performance reviewer ran. This diff touches an ownership check, so a deeper pass with /ce-code-review is warranted.
```

## Heading

`## Quick Review -- <N> action items, <M> blocking`, where `M` counts the `Fix before merge` bucket. Drop the `, <M> blocking` clause when `M` is 0. With no items at all, use `## Quick Review -- no action items` and one plain line naming what ran: three reviewers found nothing actionable, which is not the same as a clean bill of health.

## Item body

One to three sentences, in this order:

1. **What breaks** -- the finding's `why_it_matters`, in terms of observable behavior: a wrong result, an error path, a contract mismatch, an undetected regression.
2. **The fix** -- the finding's `suggested_fix`, as a concrete instruction. Name the symbol, guard, or test to add. "Consider refactoring" is not an action; if no action can be named, the item does not ship.

When a finding rests on an unresolved boundary from `residual_risks` (for example, callsite completeness established by grep only), say so in one clause inside the body rather than in a separate section.

## Notes rule

`### Notes` exists only for facts that change whether the items above apply or whether the list can be trusted. At most three lines, one line each:

- A **past learning** that applies to a specific item (reference it by `#`) or that names a documented pattern the diff repeats. A learning that produces no action is dropped, not reported.
- A **scope caveat** — the local tree is behind the remote branch, the diff base was overridden, a reviewer's return was dropped as malformed so its concern went unreviewed.
- A **missing-reviewer routing line** when the diff turns out to warrant reviewers this roster does not have (auth, migrations, public API contracts, CI/deploy gates): one line recommending `/ce-code-review`. Use `$ce-code-review` when the active harness is Codex or otherwise documents dollar-prefixed invocation. Output exactly one form.
- An **open question for the author** that no code change resolves.

Do not use Notes for suppressed-finding counts, reviewer rosters, residual-risk inventories, or a restatement of the items.
