---
name: ce-agent-review
description: Invoke a single named reviewer persona for a focused code review. Use when you want one specific style of review (e.g. Jeremy's React readability lens, Brandon's architecture lens, or a security-only pass) instead of the full ce-code-review panel.
argument-hint: "<reviewer-name> [optional: file path or MR/PR number]"
allowed-tools: Bash(git *), Bash(glab *), Bash(gh *), Read, Grep, Glob
---

# Agent Review

Run a single reviewer persona against a diff, file, or merge/pull request. This is the focused, one-lens counterpart to `ce-code-review` (which spawns the full right-sized panel). Use it when you already know which perspective you want.

## Usage

```bash
/agent-review jeremy                    # Jeremy Gillick's React readability lens on current changes
/agent-review brandon src/App.tsx       # Brandon Aldrich's architecture lens on a file
/agent-review security 123              # Security-only review of MR/PR !123
```

## Reviewer roster

Each name maps to a review lens. This skill dispatches a generic subagent seeded with the lens summary below (kept self-contained here so the skill works standalone). The "Lens" column names the corresponding full persona used by `ce-code-review`, for reference.

| Name | Lens | Focus |
|------|------|-------|
| `jeremy` | jeremy-gillick-reviewer | React readability: extract inline logic, `useMemo`/`useCallback`, duplication, clear naming, test organization. Educational tone, `praise:`/`nit:` prefixes. |
| `brandon` | brandon-aldrich-reviewer | Architecture: module boundaries, batched-query performance, component simplicity, references to codebase patterns. Request/Question/Praise prefixes. |
| `julik` | julik-frontend-races-reviewer | Async UI, Stimulus/Turbo lifecycles, DOM-timing races, janky failure modes. |
| `security` | security-reviewer | Exploitable vulnerabilities in auth, public endpoints, user input, permission checks. |
| `performance` | performance-reviewer | DB queries, loop-heavy transforms, caching, I/O-intensive paths, scalability. |
| `correctness` | correctness-reviewer | Logic errors, edge cases, state bugs, error propagation, intent-vs-implementation. |
| `maintainability` | maintainability-reviewer | Premature abstraction, indirection, dead code, coupling, naming. |
| `testing` | testing-reviewer | Coverage gaps, weak assertions, brittle implementation-coupled tests. |
| `reliability` | reliability-reviewer | Error handling, retries, timeouts, background jobs, async handlers. |
| `api-contract` | api-contract-reviewer | Breaking contract changes: routes, request/response types, serialization, versioning. |
| `data-migration` | data-migration-reviewer | Migration/backfill safety, ID mappings, column renames, enum conversions. |
| `deployment` | deployment-verification-agent | Go/No-Go checklists, SQL verification, rollback procedures, monitoring. |
| `agent-native` | agent-native-reviewer | Action parity -- any action a user can take, an agent can too. |
| `project-standards` | project-standards-reviewer | CLAUDE.md / AGENTS.md compliance: frontmatter, references, naming, portability. |

## Implementation

1. **Parse arguments.** First token = reviewer name (required). Remaining tokens = scope (file path, MR/PR number, `.` for whole codebase, or empty for uncommitted changes).

2. **Validate the reviewer name** against the roster. On an unknown or missing name, print the roster and the usage line, then stop.

3. **Resolve scope and gather the diff/context:**
   ```bash
   git diff                       # no scope -> uncommitted changes
   git diff <base>...HEAD         # branch changes
   glab mr diff <number>          # GitLab MR
   gh pr diff <number>            # GitHub PR
   ```
   For a file path, read the file(s). For `.`, warn that a whole-codebase review is broad and confirm scope.

4. **Dispatch one generic subagent** seeded with the selected lens from the roster above. Instruct it to review only the gathered diff/context through that lens, and to report findings with concrete file:line references and actionable suggestions in the reviewer's style. Do not dispatch a standalone agent by registered type/name -- seed a generic subagent with the lens description.

5. **Present the review** to the user, grouped by severity, with each finding tied to a location.

## Notes

- For a comprehensive multi-lens review, use `ce-code-review` instead -- it right-sizes the full panel automatically.
- Rails/Ruby/Python-specific reviewer personas are intentionally absent in this fork.
