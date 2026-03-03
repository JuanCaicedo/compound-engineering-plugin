---
name: generate-review-guide
description: Generates comprehensive reviewer's guides from GitLab merge request URLs, including architecture overview, testing checklists, and critical areas to review. Use when the user provides a GitLab MR URL or asks to "create a review guide", "generate reviewer's guide", "review this MR", or "analyze this merge request".
---

# Generating MR Reviews

Creates detailed, actionable reviewer's guides from GitLab merge requests to help reviewers understand changes, test effectively, and provide thorough feedback.

## Quick Start

**Input:** GitLab MR URL
**Output:** Comprehensive markdown review guide saved to `~/code/vanna/docs/reviews/`

```bash
User: "Generate a reviewer's guide for https://gitlab.com/org/repo/-/merge_requests/123"
```

The skill will:
1. Fetch MR details and changes
2. Analyze architecture and files
3. Generate structured review guide
4. Save to `~/code/vanna/docs/reviews/MR-{number}-{slug}.md`

## Instructions

### Step 1: Extract MR Information

Parse the GitLab URL to get:
- Repository path (org/repo)
- MR number
- Use `glab mr view {number} --repo {repo}` to fetch:
  - Title, author, description
  - Status, labels, reviewers
  - Comments and discussion

### Step 2: Analyze Changes

Use `glab mr diff {number} --repo {repo}` to:
- List all changed files
- Count additions/deletions (`git diff --stat`)
- Identify new vs modified files
- Group files by type (frontend, backend, database, tests, config)

### Step 3: Understand Key Files

For major changes, use `git show origin/{branch}:{filepath}` to read:
- Main page/component files
- New API endpoints or routes
- Database migrations
- Configuration changes
- i18n files for feature scope

Read 3-5 most critical files to understand:
- Architecture patterns used
- Data flow
- Component hierarchy
- API contracts

### Step 4: Generate Review Guide

Use the template below, customizing sections based on MR type:

**Always include:**
- Overview with MR links
- Key changes summary
- Architecture diagram or file tree
- Testing checklist
- Files prioritized by review importance

**Include when relevant:**
- Database migration review points (if schema changes)
- Performance considerations (if queries/large datasets)
- Security checklist (if auth/permissions/data handling)
- Mobile responsive testing (if UI changes)
- API contract review (if backend changes)
- i18n coverage (if user-facing text)

**Customize for MR type:**
- **Feature addition**: Focus on architecture, testing scenarios, edge cases
- **Bug fix**: Focus on root cause, regression prevention, test coverage
- **Refactor**: Focus on behavior preservation, migration path, rollback
- **Performance**: Focus on benchmarks, metrics, monitoring
- **Config/Infrastructure**: Focus on deployment, rollback, monitoring

### Step 5: Save the File

Create filename: `MR-{number}-{slug}.md`
- Extract slug from MR title (lowercase, hyphens, max 40 chars)
- Save to `~/code/vanna/docs/reviews/`
- Create directory if needed

## Review Guide Template

```markdown
# Reviewer's Guide: {MR Title}

**MR**: [!{number}]({url})
**Branch**: `{branch-name}`
**Author**: {author}
**Linear**: [{ticket-id}]({linear-url}) _(if linked)_

## Overview
{1-2 sentence description of what this MR does}

**Demo**: [Loom Video]({video-url}) _(if provided)_

## Key Changes Summary

### 📊 New Features / 🐛 Bug Fixes / ♻️ Refactor
{Bullet list of main changes}

### 📁 Files Changed
- **{count} files** changed
- **+{additions} / -{deletions} lines**

---

## Architecture & Design

{Show file tree or component diagram}

### Frontend / Backend / Database
{Relevant sections based on changes}

---

## What to Review

### 🎯 Critical Areas

#### 1. {Critical Area Name}
Files: {relevant files}
- [ ] {Review checkpoint}
- [ ] {Review checkpoint}

### 🧪 Testing Scenarios

#### Happy Path
1. {Test step}
2. {Test step}

#### Edge Cases
1. {Edge case scenario}
2. {Edge case scenario}

#### Error Handling
1. {Error scenario}
2. {Error scenario}

### 🔍 Code Quality

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

- **Small MRs (<10 files)**: Read all changed files
- **Medium MRs (10-50 files)**: Read critical files, skim others
- **Large MRs (>50 files)**: Focus on new files and major changes, note file count

### Prioritization

Files to **always read** if present:
- Database migrations
- API route definitions
- Main page/feature entry points
- Package.json changes (dependencies)

Files to **always mention** but can skip reading:
- Auto-generated files (routeTree.ts, schema.ts)
- Test files (mention coverage exists)
- Lock files

### Tone

- Be specific and actionable
- Focus on "what to check" not "what they did"
- Provide concrete test scenarios
- Ask clarifying questions when logic is complex
- Highlight risks without being alarmist

### Common Sections by Type

**Database migrations:**
- Index performance impact
- Migration reversibility
- Data transformation correctness
- Production deployment timing

**API changes:**
- Type safety
- Input validation
- Error handling
- Permission checks
- Response shape consistency

**UI changes:**
- Responsive design
- Loading states
- Error states
- Empty states
- Accessibility

**Configuration:**
- Feature flag strategy
- Rollback plan
- Environment parity
- Secrets handling

## Examples

### Example 1: Feature Addition

**Input:**
```
User: "Generate review guide for https://gitlab.com/vanna/connect/-/merge_requests/1929"
```

**Output:** Creates `MR-1929-referrals-home.md` with:
- Overview of new referrals dashboard
- Architecture showing component hierarchy
- Testing checklist for stats, filters, drawers
- Database migration review section
- Performance testing guidance
- Questions about caching, permissions

### Example 2: Bug Fix

**Input:**
```
User: "Review guide for MR 1845"
```

**Output:** Creates `MR-1845-fix-patient-search.md` with:
- Root cause explanation
- Files changed (minimal)
- Regression test checklist
- Related code to verify
- Questions about edge cases

### Example 3: Quick Review Request

**Input:**
```
User: "Can you review https://gitlab.com/vanna/connect/-/merge_requests/2001?"
```

**Output:** Generates full guide even though user said "review" not "generate guide"

## Advanced Features

### Fetching Branch Code

To read files from the MR branch (when not checked out):

```bash
# Fetch branch
git fetch origin {branch}:refs/remotes/origin/{branch}

# Read file from branch
git show origin/{branch}:typescript/path/to/file.tsx
```

### Multiple MRs

Process multiple MRs in sequence:

```bash
User: "Generate reviews for MR 1929, 1930, and 1931"
```

Create separate files for each.

### Custom Output Location

If user specifies different path:

```bash
User: "Generate review guide for MR 1929 in ~/Desktop/"
```

Save to specified location instead of default.

## Related Skills

- `compound-engineering:review` - For comprehensive multi-agent code review
- `compound-engineering:plan_review` - For reviewing implementation plans

## Success Criteria

A good review guide:
- ✅ Has clear overview linking to MR and Linear
- ✅ Breaks down architecture visually
- ✅ Provides actionable test scenarios
- ✅ Prioritizes files by review importance
- ✅ Asks clarifying questions for author
- ✅ Includes deployment considerations
- ✅ Saves to correct location with clear filename
