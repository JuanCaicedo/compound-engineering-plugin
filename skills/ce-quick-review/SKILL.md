---
name: ce-quick-review
description: "Lightweight code review with 3 focused agents -- correctness, quality, and learnings. Use for fast feedback when ce:review is too heavy."
argument-hint: "[blank to review current branch, or base:<ref>]"
---

# Quick Review

Lightweight code review that dispatches exactly 3 agents in parallel: correctness, quality (combined maintainability + testing), and learnings researcher. Produces a single findings report with confidence gating.

## When to Use

- Fast feedback before creating a PR
- When `/ce:review` is too heavy for a small or medium change
- During iterative development for quick sanity checks
- For changes that don't need security, performance, or stack-specific reviewers

For comprehensive reviews with conditional reviewers, plan verification, and fix automation, use `/ce:review` instead.

## Argument Parsing

Parse `$ARGUMENTS` for the following optional tokens. Strip each recognized token before interpreting the remainder.

| Token | Example | Effect |
|-------|---------|--------|
| `mode:headless` | `mode:headless` | Structured text output for programmatic callers |
| `base:<sha-or-ref>` | `base:abc1234` or `base:origin/main` | Use this as the diff base directly |

All tokens are optional. When absent, review the current branch against its base.

## Mode Detection

| Mode | When | Behavior |
|------|------|----------|
| **Interactive** (default) | No mode token | Review and present findings. No file edits. |
| **Headless** | `mode:headless` | Return structured text envelope for programmatic callers. No interactive prompts. |

## Severity Scale

| Level | Meaning |
|-------|---------|
| **P0** | Critical breakage, exploitable vulnerability, data loss/corruption |
| **P1** | High-impact defect likely hit in normal usage |
| **P2** | Moderate issue with meaningful downside (edge case, maintainability trap) |
| **P3** | Low-impact, narrow scope, minor improvement |

## Reviewers

Fixed 3-agent team. No conditional selection.

| Agent | Focus |
|-------|-------|
| `compound-engineering:review:correctness-reviewer` | Logic errors, edge cases, state bugs, error propagation |
| `compound-engineering:review:quality-reviewer` | Maintainability + testing (naming, abstraction, dead code, coverage gaps, brittle tests) |
| `compound-engineering:research:learnings-researcher` | Search docs/solutions/ for past issues related to this change |

## How to Run

### Stage 1: Determine scope

Compute the diff range, file list, and diff content.

**If `base:` argument is provided (fast path):**

Use the provided value directly:

```
BASE_ARG="{base_arg}"
BASE=$(git merge-base HEAD "$BASE_ARG" 2>/dev/null) || BASE="$BASE_ARG"
```

Then produce the diff:

```
echo "BASE:$BASE" && echo "FILES:" && git diff --name-only $BASE && echo "DIFF:" && git diff -U10 $BASE
```

**If no argument (standalone on current branch):**

Detect the review base branch using `references/resolve-base.sh`:

```
RESOLVE_OUT=$(bash references/resolve-base.sh) || { echo "ERROR: resolve-base.sh failed"; exit 1; }
if [ -z "$RESOLVE_OUT" ] || echo "$RESOLVE_OUT" | grep -q '^ERROR:'; then echo "${RESOLVE_OUT:-ERROR: resolve-base.sh produced no output}"; exit 1; fi
BASE=$(echo "$RESOLVE_OUT" | sed 's/^BASE://')
```

If the script outputs an error, stop. Do not fall back to `git diff HEAD`.

On success, produce the diff:

```
echo "BASE:$BASE" && echo "FILES:" && git diff --name-only $BASE && echo "DIFF:" && git diff -U10 $BASE
```

Extract the base marker, file list, and diff content from the output.

**If `mode:headless` and scope cannot be determined** (no branch, no `base:` ref), emit `Review failed (headless mode). Reason: no diff scope detected. Re-invoke with base:<ref>.` and stop.

### Stage 2: Intent discovery

Extract intent from the branch name and commit messages:

```
echo "BRANCH:" && git rev-parse --abbrev-ref HEAD && echo "COMMITS:" && git log --oneline ${BASE}..HEAD
```

Write a 2-3 line intent summary:

```
Intent: Simplify tax calculation by replacing the multi-tier rate lookup
with a flat-rate computation.
```

Pass this to every reviewer. Intent shapes how hard each reviewer looks.

**When intent is ambiguous:**
- **Interactive mode:** Ask one question using the platform's interactive question tool (AskUserQuestion in Claude Code, request_user_input in Codex, ask_user in Gemini): "What is the primary goal of these changes?"
- **Headless mode:** Infer conservatively from branch name and diff. Note uncertainty in Coverage.

### Stage 3: Spawn agents

Announce the team:

```
Review team:
- correctness
- quality
- learnings-researcher
```

Dispatch all 3 agents in parallel using the platform's agent/task tool. Use the mid-tier model for all agents (in Claude Code, pass `model: "sonnet"` in the Agent tool call).

Omit the `mode` parameter when dispatching sub-agents so the user's configured permission settings apply.

**Persona agents (correctness, quality):** Spawn using the subagent template included below. Each receives:

1. Their persona file content
2. Shared diff-scope rules from `references/diff-scope.md`
3. The JSON output contract from `references/findings-schema.json`
4. Review context: intent summary, file list, diff

Each agent returns full JSON (all schema fields including why_it_matters and evidence) directly to the orchestrator.

Persona sub-agents are **read-only**: they review and return structured JSON. They do not edit project files. Non-mutating inspection commands (git diff, git show, git blame, git log) are permitted.

**Learnings researcher:** Dispatch as a standard Agent call with the same review context (intent summary, file list, diff). Its output is unstructured markdown, synthesized separately in Stage 5.

**Sequential fallback:** If the platform does not support parallel dispatch, spawn agents sequentially in the order: correctness, quality, learnings-researcher.

### Stage 4: Merge findings

Process the 2 JSON returns (correctness + quality) into one deduplicated finding set.

1. **Validate.** Check each return for required top-level fields (reviewer, findings, residual_risks, testing_gaps) and per-finding fields. Drop malformed returns or findings. Record drop count.
2. **Confidence gate.** Suppress findings below 0.60 confidence. Exception: P0 findings at 0.50+ survive. Record suppressed count.
3. **Deduplicate.** Compute fingerprint: `normalize(file) + line_bucket(line, +/-3) + normalize(title)`. When fingerprints match, merge: keep highest severity, keep highest confidence, note which reviewers flagged it.
4. **Separate pre-existing.** Pull out findings with `pre_existing: true` into a separate list.
5. **Sort.** Order by severity (P0 first) -> confidence (descending) -> file path -> line number.
6. **Collect coverage data.** Union residual_risks and testing_gaps across both reviewers.

### Stage 5: Synthesize and present

Assemble the report using **pipe-delimited markdown tables** from the review output template included below.

1. **Header.** Scope, intent, reviewer team.
2. **Findings.** Pipe-delimited tables grouped by severity (`### P0 -- Critical`, `### P1 -- High`, `### P2 -- Moderate`, `### P3 -- Low`). Omit empty severity levels.
3. **Pre-existing.** Separate section, does not count toward verdict.
4. **Learnings & Past Solutions.** Surface learnings-researcher results. Omit if no relevant learnings found.
5. **Coverage.** Suppressed count, residual risks, testing gaps.
6. **Verdict.** Ready to merge / Ready with fixes / Not ready. Fix order if applicable.

**Format verification:** Before delivering the report, verify findings use pipe-delimited table rows, not freeform text.

### Headless output format

In `mode:headless`, replace the interactive tables with a structured text envelope:

```
Quick review complete (headless mode).

Scope: <scope-line>
Intent: <intent-summary>
Reviewers: correctness, quality, learnings-researcher
Verdict: <Ready to merge | Ready with fixes | Not ready>

[P1][gated_auto -> downstream-resolver] File: <file:line> -- <title> (correctness, confidence <N>)
  Why: <why_it_matters>
  Evidence: <evidence[0]>

[P2][advisory -> human] File: <file:line> -- <title> (quality, confidence <N>)
  Why: <why_it_matters>

Pre-existing issues:
[P2][advisory -> human] File: <file:line> -- <title> (correctness, confidence <N>)

Learnings & Past Solutions:
- <learning>

Coverage:
- Suppressed: <N> findings below 0.60 confidence

Review complete
```

Omit any section with zero items. End with "Review complete" as the terminal signal.

## Quality Gates

Before delivering the report, verify:

1. **Every finding is actionable.** If it says "consider" or "might want to" without a concrete fix, rewrite it with a specific action.
2. **No false positives from skimming.** Verify the surrounding code was actually read.
3. **Severity is calibrated.** A style nit is never P0. A data loss bug is never P3.
4. **Line numbers are accurate.** Verify each cited line number against the file content.

@./references/subagent-template.md
@./references/diff-scope.md
@./references/findings-schema.json
@./references/review-output-template.md
