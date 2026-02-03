# Quick Review Output Template

Use this **exact format** when presenting synthesized review findings. Findings are grouped by severity, not by reviewer.

**IMPORTANT:** Use pipe-delimited markdown tables (`| col | col |`). Do NOT use ASCII box-drawing characters.

## Example

```markdown
## Quick Review Results

**Scope:** merge-base with the review base branch -> working tree (14 files, 342 lines)
**Intent:** Add order export endpoint with CSV and JSON format support

**Reviewers:** correctness, quality, learnings-researcher

### P0 -- Critical

| # | File | Issue | Reviewer | Confidence | Route |
|---|------|-------|----------|------------|-------|
| 1 | `orders_controller.rb:42` | User-supplied ID in account lookup without ownership check | correctness | 0.92 | `gated_auto -> downstream-resolver` |

### P1 -- High

| # | File | Issue | Reviewer | Confidence | Route |
|---|------|-------|----------|------------|-------|
| 2 | `export_service.rb:87` | Missing error handling for CSV serialization failure | correctness | 0.85 | `safe_auto -> review-fixer` |
| 3 | `export_service.rb:91` | Behavioral change with no test additions | quality | 0.80 | `manual -> downstream-resolver` |

### P2 -- Moderate

| # | File | Issue | Reviewer | Confidence | Route |
|---|------|-------|----------|------------|-------|
| 4 | `export_helper.rb:12` | Unnecessary wrapper class -- single pass-through to underlying method | quality | 0.70 | `advisory -> human` |

### Pre-existing Issues

| # | File | Issue | Reviewer |
|---|------|-------|----------|
| 1 | `orders_controller.rb:12` | Broad rescue masking failed permission check | correctness |

### Learnings & Past Solutions

- [Known Pattern] `docs/solutions/export-pagination.md` -- previous export pagination fix applies to this endpoint

### Coverage

- Suppressed: 2 findings below 0.60 confidence

---

> **Verdict:** Ready with fixes
>
> **Reasoning:** 1 critical auth bypass must be fixed. Test coverage gap (P1) should be addressed.
>
> **Fix order:** P0 auth bypass -> P1 error handling + tests
```

## Formatting Rules

- **Pipe-delimited markdown tables** for findings -- never ASCII box-drawing characters
- **Severity-grouped sections** -- `### P0 -- Critical`, `### P1 -- High`, `### P2 -- Moderate`, `### P3 -- Low`. Omit empty severity levels.
- **Always include file:line location** for code review issues
- **Reviewer column** shows which persona(s) flagged the issue. Multiple reviewers = cross-reviewer agreement.
- **Route column** shows `<autofix_class> -> <owner>`
- **Header includes** scope, intent, and reviewer team
- **Pre-existing section** -- separate table, no confidence column (informational only)
- **Learnings section** -- results from learnings-researcher, with links to docs/solutions/ files. Omit if no relevant learnings found.
- **Coverage section** -- suppressed count, residual risks, testing gaps
- **Verdict uses blockquotes** with reasoning and fix order
- **Horizontal rule** (`---`) separates findings from verdict
