---
name: ce-vanna-patterns-review
description: "Review current branch changes against documented patterns and best practices in the Vanna docs repository. Use when finishing a feature branch and wanting to check for pattern violations before opening a merge request."
argument-hint: "[base-branch]"
---

# Vanna Patterns Review

<command_purpose>Evaluate the current branch's changes against the team's documented patterns, best practices, and architectural decisions in the Vanna docs repository. Produces a prioritized triage list of findings.</command_purpose>

## Introduction

<role>Senior engineer who has internalized every documented pattern, best practice, and architectural decision in the team's knowledge base. Review code changes with that institutional context, flagging violations and surfacing relevant guidance.</role>

## Prerequisites

<requirements>
- Working in a Git repository with changes on a feature branch
- The Vanna docs repository is available locally (see Locating the Docs Repository)
- Current branch has commits diverged from the base branch
</requirements>

## Locating the Docs Repository

This skill reads a docs repository that lives outside the current project. Resolve its path once, at the start of the run, in this order:

1. `$VANNA_DOCS_ROOT`, when set and it contains a `solutions/` directory.
2. The conventional checkout location, `$HOME/code/vanna/docs`.

Store the result as `DOCS_ROOT` and use it for every read below. If neither location resolves, tell the user the docs repository could not be found, name both locations you tried, and stop — a pattern review with no patterns loaded is worse than no review, because it reads as a clean bill of health.

## Workflow

### Phase 0: Determine Scope

<task_list>

- [ ] Determine the base branch: use `$ARGUMENTS` if provided, otherwise default to `main`
- [ ] Get the current branch name: `git branch --show-current`
- [ ] Get the list of changed files: `git diff <base>...HEAD --name-only`
- [ ] Get the full diff: `git diff <base>...HEAD`
- [ ] If no changes are found, inform the user and stop

</task_list>

### Phase 1: Load Relevant Patterns

<thinking>
Based on the changed files and diff content, determine which categories of documented patterns are relevant. Load only what matters — don't read the entire docs repo.
</thinking>

<task_list>

- [ ] **Always read** the critical patterns file: `$DOCS_ROOT/solutions/patterns/critical-patterns.md`
- [ ] **Identify relevant modules** from changed file paths (e.g., `app/components/member_record/` → MemberRecord module)
- [ ] **Identify change types** from the diff:

| Change Type | Docs to Load (relative to `$DOCS_ROOT`) |
|-------------|-------------|
| React components, JSX, TSX | `solutions/best-practices/`, `solutions/ui-patterns/`, `solutions/architecture-decisions/` |
| Hooks, queries, data fetching | `solutions/best-practices/`, `solutions/architecture-decisions/` |
| Tests, specs | `solutions/test-failures/`, `solutions/best-practices/` (testing-related) |
| Database, migrations | `solutions/database-issues/` |
| Styling, CSS, layout | `solutions/ui-patterns/` |
| API, integrations | `solutions/integration-issues/` |
| General logic | `solutions/logic-errors/`, `solutions/best-practices/` |

- [ ] **Grep for module-specific docs** under `$DOCS_ROOT/solutions/`, matching on frontmatter `module:`, `tags:`, and `title:` fields for the module names and keywords drawn from the changed paths. Run these searches in parallel, then combine and deduplicate the results.

- [ ] **Read frontmatter** (first 30 lines) of each candidate file to confirm relevance
- [ ] **Fully read** only the files that are clearly relevant to the changes (strong or moderate match)

</task_list>

### Phase 2: Evaluate Changes Against Patterns

<thinking>
For each loaded pattern/best-practice, check whether the diff introduces code that violates or misaligns with the documented guidance. Be specific — cite the pattern document and the offending line(s) in the diff.
</thinking>

Go through each relevant pattern document and check the diff for:

1. **Direct violations** — Code that contradicts a documented pattern (e.g., using spread props when docs say to use explicit props)
2. **Missing patterns** — Code that should follow a documented pattern but doesn't (e.g., a new component that doesn't use container/presentational split when it should)
3. **Anti-patterns** — Code that matches a documented "WRONG" example
4. **Opportunities** — Places where a documented best practice could improve the code but isn't strictly violated

For each finding, record:
- The pattern document (path and title)
- The specific code in the diff that triggers the finding
- The severity (see Phase 3)
- A concrete suggestion for how to fix it

Only report a finding you can tie to a specific documented pattern and a specific line in the diff. Do not pad the list with generic code-review advice — that is what `ce-code-review` is for, and mixing the two makes it unclear which findings carry the team's documented authority.

### Phase 3: Compile Triage List

<critical_requirement>Present findings as a single, prioritized triage list. Group by severity. Be actionable — each item should tell the developer exactly what to change and why.</critical_requirement>

#### Severity Levels

| Level | Meaning | Criteria |
|-------|---------|----------|
| **P1 — Must Fix** | Violates a critical pattern or documented architectural decision | Pattern is in `critical-patterns.md`, or is an architecture decision, or has severity: critical/high |
| **P2 — Should Fix** | Violates a documented best practice | Pattern is a best-practice with clear WRONG/CORRECT examples |
| **P3 — Consider** | Opportunity to apply a documented pattern | Pattern is relevant but the code isn't strictly wrong |

#### Output Format

Present findings in this format:

```markdown
## Vanna Patterns Review

**Branch:** [branch-name]
**Base:** [base-branch]
**Files Changed:** [count]
**Patterns Checked:** [count of docs loaded]

---

### P1 — Must Fix ([count])

#### 1. [Short description]
- **Pattern:** [Title from doc] (`solutions/[path]`)
- **File:** `[changed-file]:[line-range]`
- **Issue:** [What the code does wrong]
- **Fix:** [Concrete suggestion]

---

### P2 — Should Fix ([count])

#### 1. [Short description]
- **Pattern:** [Title from doc] (`solutions/[path]`)
- **File:** `[changed-file]:[line-range]`
- **Issue:** [What the code does wrong]
- **Fix:** [Concrete suggestion]

---

### P3 — Consider ([count])

#### 1. [Short description]
- **Pattern:** [Title from doc] (`solutions/[path]`)
- **File:** `[changed-file]:[line-range]`
- **Opportunity:** [How applying this pattern would improve the code]

---

### Patterns Consulted

| Document | Category | Relevant? |
|----------|----------|-----------|
| [title] | [category] | Yes/No — [brief reason] |

### No Issues Found
[If clean, explicitly state: "No pattern violations found. The changes align with documented practices."]
```

### Phase 4: Offer Next Steps

After presenting the triage list, offer:

1. **Fix P1 items now** — Address must-fix items immediately
2. **Track the rest** — Record the remaining findings as todos so they survive this session
3. **Done** — Acknowledge the review and move on

If the user chooses to fix items, work through them one at a time, verifying each fix against the original pattern document.
