---
name: ce-generate-review-guide
description: Generates comprehensive reviewer's guides from GitLab merge request URLs, including architecture overview, testing checklists, and critical areas to review. Use when the user provides a GitLab MR URL or asks to "create a review guide", "generate reviewer's guide", or "analyze this merge request".
argument-hint: "[GitLab MR URL or MR number]"
---

# Generate MR Reviewer's Guide

Creates detailed, actionable reviewer's guides from GitLab merge requests to help reviewers understand changes, test effectively, and provide thorough feedback.

This skill produces a guide *for a human reviewer to work from*. It does not itself judge the code — for findings, use `ce-code-review`.

## Platform

GitLab only. This skill uses `glab`. Confirm the repo is GitLab with `glab repo view` before fetching; if that fails because the host is GitHub, say so and stop rather than issuing `glab` calls that will error confusingly.

## Output Location

Guides are written outside the reviewed project, to a central docs repository. Resolve the destination once, at the start of the run, in this order:

1. An explicit path in the user's request ("…in ~/Desktop"), which always wins.
2. `$VANNA_DOCS_ROOT/reviews`, when `$VANNA_DOCS_ROOT` is set.
3. The conventional checkout location, `$HOME/code/vanna/docs/reviews`.

Create the directory if it does not exist. Store the result as `REVIEWS_DIR`.

## Workflow

### Step 1: Extract MR Information

Parse the GitLab URL to get the repository path (`org/repo`) and the MR number. Then fetch details:

```bash
glab mr view MR_NUMBER --repo ORG/REPO
```

Collect title, author, description, status, labels, reviewers, and discussion.

### Step 2: Analyze Changes

```bash
glab mr diff MR_NUMBER --repo ORG/REPO
```

From the diff: list all changed files, count additions/deletions, identify new vs. modified files, and group files by type (frontend, backend, database, tests, config).

### Step 3: Understand Key Files

For major changes, read the file as it exists on the MR branch:

```bash
git fetch origin BRANCH:refs/remotes/origin/BRANCH
git show origin/BRANCH:path/to/file.tsx
```

Read the 3-5 most critical files to understand architecture patterns, data flow, component hierarchy, and API contracts.

Files to **always read** if present: database migrations, API route definitions, main page/feature entry points, and dependency manifest changes.

Files to **always mention** but skip reading: auto-generated files (route trees, generated schemas), test files (note that coverage exists), and lock files.

### Step 4: Generate the Guide

Use the template below, customizing sections based on MR type.

**Always include:** overview with MR links, key changes summary, architecture diagram or file tree, testing checklist, and files prioritized by review importance.

**Include when relevant:**
- Database migration review points (if schema changes)
- Performance considerations (if queries/large datasets)
- Security checklist (if auth/permissions/data handling)
- Mobile responsive testing (if UI changes)
- API contract review (if backend changes)
- i18n coverage (if user-facing text)

**Customize for MR type:**
- **Feature addition**: architecture, testing scenarios, edge cases
- **Bug fix**: root cause, regression prevention, test coverage
- **Refactor**: behavior preservation, migration path, rollback
- **Performance**: benchmarks, metrics, monitoring
- **Config/Infrastructure**: deployment, rollback, monitoring

### Step 5: Save the File

Filename: `MR-{number}-{slug}.md`, where the slug comes from the MR title (lowercase, hyphens, max 40 chars). Save into `REVIEWS_DIR`.

When the user asks for several MRs at once, generate a separate file for each, in sequence.

## Review Guide Template

```markdown
# Reviewer's Guide: {MR Title}

**MR**: [!{number}]({url})
**Branch**: `{branch-name}`
**Author**: {author}
**Ticket**: [{ticket-id}]({ticket-url}) _(if linked)_

## Overview
{1-2 sentence description of what this MR does}

**Demo**: [Video]({video-url}) _(if provided)_

## Key Changes Summary

### New Features / Bug Fixes / Refactor
{Bullet list of main changes}

### Files Changed
- **{count} files** changed
- **+{additions} / -{deletions} lines**

---

## Architecture & Design

{Show file tree or component diagram}

### Frontend / Backend / Database
{Relevant sections based on changes}

---

## What to Review

### Critical Areas

#### 1. {Critical Area Name}
Files: {relevant files}
- [ ] {Review checkpoint}
- [ ] {Review checkpoint}

### Testing Scenarios

#### Happy Path
1. {Test step}
2. {Test step}

#### Edge Cases
1. {Edge case scenario}
2. {Edge case scenario}

#### Error Handling
1. {Error scenario}
2. {Error scenario}

### Code Quality

#### Components/Modules to Review
1. **{filename}** ({line count} lines)
   - [ ] {Review point}
   - [ ] {Review point}

---

## Files to Focus On

### Must Review (Priority 1)
1. `{filepath}` - {reason}
2. `{filepath}` - {reason}

### Should Review (Priority 2)
{Additional important files}

### Can Skim (Priority 3)
{Generated/boilerplate files}

---

## Questions for Author

1. **{Topic}**: {Question}
2. **{Topic}**: {Question}

---

## Testing Checklist

### Setup
- [ ] {Setup step}

### Functional Testing
- [ ] {Test case}
- [ ] {Test case}

### Performance
- [ ] {Performance check}

---

## Deployment Considerations

1. **{Consideration}**: {Details}
2. **{Consideration}**: {Details}

---

## Notes

Generated on {date}
```

## Guidelines

### Analysis Depth

- **Small MRs (<10 files)**: read all changed files
- **Medium MRs (10-50 files)**: read critical files, skim others
- **Large MRs (>50 files)**: focus on new files and major changes, and state the file count so the reviewer knows the guide is partial

### Tone

- Be specific and actionable
- Focus on "what to check", not "what they did"
- Provide concrete test scenarios
- Ask clarifying questions when logic is complex
- Highlight risks without being alarmist

### Common Sections by Type

**Database migrations:** index performance impact, migration reversibility, data transformation correctness, production deployment timing.

**API changes:** type safety, input validation, error handling, permission checks, response shape consistency.

**UI changes:** responsive design, loading states, error states, empty states, accessibility.

**Configuration:** feature flag strategy, rollback plan, environment parity, secrets handling.

## Success Criteria

- Clear overview linking to the MR and its ticket
- Architecture broken down visually
- Actionable test scenarios, not restated diff
- Files prioritized by review importance
- Clarifying questions raised for the author
- Deployment considerations covered
- Saved to `REVIEWS_DIR` with a clear filename
