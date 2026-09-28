## Code Review -- 3 action items, 2 blocking

**Scope:** merge-base with main -> working tree
**Intent:** Demonstrate stable finding numbering
**Mode:** markdown + explicit local apply
**Reviewers:** correctness, testing, maintainability
**Artifacts:** `/tmp/compound-engineering-501/ce-code-review/20260821-142233/`

### Notes

- The `maintainability` reviewer returned no findings on this diff.

### Applied

| # | File | Fix | Reviewer |
|---|------|-----|----------|
| 4 | `export_service.rb:60 (+test)` | Tightened export file perms 0644 -> 0600 (security-posture — verify in diff) | security |

Validation: tests 18 -> 19; suite 96 pass, lint clean.

### Triage Groups

| Group | Findings | Context | Preferred Resolution | Why |
|-------|----------|---------|----------------------|-----|
| Export result-set scaling | #1, #2 | Both stem from loading the full order set in one pass; decision-gate | Decide the pagination contract (#2), then stream behind it (#1) | One cursor/page decision resolves the memory bound and the API shape together |

### Worth fixing

**3. Return a 4xx on CSV serialization failure** -- `export_service.rb:45` (P2, correctness, confidence 75, gated_auto -> downstream-resolver)
A serialization error escapes as a 500, so a caller cannot tell a bad `format` parameter from a server fault. Rescue `CSV::MalformedCSVError` around the row writer and return 422.

### Fix before merge

**2. Decide the export pagination contract** -- `export_service.rb:91` (P1, api-contract, confidence 75, manual -> downstream-resolver, decision)
The endpoint returns every row in one response, so the shape cannot change after GA without breaking clients. Choose a cursor or page/limit contract before #1 lands, since it fixes the streaming boundary.

**1. Stream the export result set instead of materializing it** -- `export_service.rb:87` (P1, performance, confidence 100, gated_auto -> downstream-resolver)
`Order.where(...).to_a` materializes the full result set, so a large account OOMs the worker. Replace it with `find_each`, or paginate behind the contract decided in #2.
