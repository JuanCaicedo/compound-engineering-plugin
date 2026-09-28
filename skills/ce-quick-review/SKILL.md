---
name: ce-quick-review
description: "Fast code review with a fixed three-agent team -- correctness, combined quality (maintainability plus testing), and past learnings -- producing one numbered action list and applying no fixes. Use when the user asks for a lightweight, three-agent, or fast pre-PR review of a small or medium local change. Use `ce-code-review` instead when the change needs diff-aware persona selection, security, performance, or stack-specific reviewers, a PR or remote target, plan verification, cross-model adversarial review, or local apply."
argument-hint: "[mode:headless] [blank to review the current branch, or base:REF]"
---

# Quick Review

Lightweight code review that dispatches exactly 3 subagents in parallel — correctness, quality (maintainability + testing combined), and a learnings researcher — then merges their output into one numbered action list: what to change, where, and why, ordered so the list doubles as the fix order. No conditional persona selection, no PR targeting, no cross-model pass, no fixes applied.

## When to Use

- Fast feedback on a small or medium change before opening a PR
- Iterative development sanity checks, where a full review is more ceremony than the change warrants
- Changes that plainly do not need security, performance, or stack-specific reviewers

**Route to `ce-code-review` instead** when the change needs diff-aware persona selection, a PR or remote target, plan/requirements verification, cross-model adversarial review, or authorized local apply. Reach for it whenever the diff touches auth, data migrations, public API contracts, or a silent-pass verification mechanism (CI gate, deploy step, coverage/lint gate) — this skill's fixed roster has no reviewer for those concerns and will not tell you it is missing one.

**Relationship to `ce-code-review`'s quick short-circuit.** `ce-code-review` recognizes a bare "quick review" ask and forwards it to the harness's built-in review. That path and this skill are different tiers: the short-circuit delegates to the harness, while this skill runs CE's own three personas against the anchored findings schema. If a run arrives here that clearly wants the harness's built-in review instead, say so in one line and stop rather than dispatching.

## Artifact root

The learnings researcher reads the project's captured learnings under `<root>/solutions/`. Resolve `<root>` once, before composing any such path, and substitute it wherever `<root>/` appears below.

- **Read** `docs_root` from `<repo-root>/.compound-engineering/config.local.yaml`, then `config.yaml`; the first non-empty value wins (`<repo-root>` is `git rev-parse --show-toplevel`). Unset means `<root>` is `docs`.
- **Validate** a set value: a repo-relative directory whose real, symlink-resolved path stays inside the repo and is neither the repo root nor under `.git/`. Otherwise stop with an error naming `docs_root` and its value — never silently fall back to `docs`.

## Argument Parsing

Parse the invocation arguments for the following optional tokens. Strip each recognized token before interpreting the remainder.

| Token | Example | Effect |
|-------|---------|--------|
| `mode:headless` | `mode:headless` | Structured text output for programmatic callers; no interactive prompts |
| `base:REF` | `base:abc1234` or `base:origin/main` | Use this ref as the diff base directly, skipping base detection |

All tokens are optional. When absent, review the current branch against its detected base.

## Mode Detection

| Mode | When | Behavior |
|------|------|----------|
| **Interactive** (default) | No mode token | Review and present findings. No file edits. |
| **Headless** | `mode:headless` | Return the structured text envelope below. No interactive prompts. |

Both modes are report-only. This skill never edits project files, commits, or pushes.

## Severity Scale

| Level | Meaning |
|-------|---------|
| **P0** | Critical breakage, exploitable vulnerability, data loss/corruption |
| **P1** | High-impact defect likely hit in normal usage |
| **P2** | Moderate issue with meaningful downside (edge case, maintainability trap) |
| **P3** | Low-impact, narrow scope, minor improvement |

## Reviewers

Fixed 3-agent team, dispatched every run. There is no conditional selection to reason about.

| Reviewer | Persona file (this skill's directory) | Focus | Model |
|----------|---------------------------------------|-------|-------|
| `correctness` | `references/personas/correctness-reviewer.md` | Logic errors, edge cases, state bugs, error propagation, intent mismatches | Session model (no override) |
| `quality` | `references/personas/quality-reviewer.md` | Maintainability + testing (naming, abstraction, dead code, coverage gaps, brittle tests) | Mid-tier |
| `learnings-researcher` | `references/personas/learnings-researcher.md` | Applicable past learnings under `<root>/solutions/` | Mid-tier |

**Model tiering is a cost guarantee, not cosmetics.** `correctness` inherits the session model because it does the highest-stakes analysis. The other two take the platform's mid-tier model at dispatch time (in Claude Code, the Sonnet class). Omitting that override on a top-tier session silently runs them at the expensive tier. In harnesses whose dispatch primitive exposes no model selector, omit the override and inherit — a working review on the parent model beats a broken dispatch on an unrecognized name.

## How to Run

### Stage 1: Determine scope

**If `base:` was provided (fast path),** resolve it directly and produce the diff:

```bash
BASE_ARG="THE_BASE_VALUE";
BASE=$(git merge-base HEAD "$BASE_ARG" 2>/dev/null) || BASE="$BASE_ARG";
echo "BASE:$BASE" && echo "FILES:" && git diff --name-only "$BASE" && echo "DIFF:" && git diff -U10 "$BASE"
```

**If no `base:` was provided,** detect the review base with the bundled resolver:

```bash
SKILL_DIR="<absolute path of the directory containing the SKILL.md you just read>";
bash "$SKILL_DIR/scripts/resolve-base.sh"
```

The resolver prints `BASE:<sha>` on success or `ERROR:<message>` on failure. **On `ERROR:` or empty output, stop and report it.** Do not fall back to `git diff HEAD` — a silently wrong base produces a review of the wrong changes, which is worse than no review.

On success, extract the sha and produce the diff:

```bash
BASE="THE_RESOLVED_SHA";
echo "FILES:" && git diff --name-only "$BASE" && echo "DIFF:" && git diff -U10 "$BASE"
```

Extract the file list and diff content from the output.

**If the diff is empty,** say so and stop — there is nothing to review.

**If `mode:headless` and scope cannot be determined** (no branch, no `base:` ref), emit `Review failed (headless mode). Reason: no diff scope detected. Re-invoke with base:REF.` and stop.

### Stage 2: Intent discovery

Extract intent from the branch name and commit messages:

```bash
BASE="THE_RESOLVED_SHA";
echo "BRANCH:" && git rev-parse --abbrev-ref HEAD && echo "COMMITS:" && git log --oneline "$BASE"..HEAD
```

Write a 2-3 line intent summary, for example:

```
Intent: Simplify tax calculation by replacing the multi-tier rate lookup
with a flat-rate computation.
```

Pass this to every reviewer. Intent shapes how hard each reviewer looks, and it is what makes intent-vs-implementation mismatches findable.

**When intent is ambiguous:**
- **Interactive mode:** ask one question through the harness's interactive question tool ("What is the primary goal of these changes?").
- **Headless mode:** infer conservatively from the branch name and diff, and carry the uncertainty into the report as a Notes line. Never block on a question in headless mode.

### Stage 3: Spawn agents

Announce the team before dispatching:

```
Review team: correctness, quality, learnings-researcher
```

Read `references/subagent-template.md` from this skill's directory now — it owns the reviewer prompt contract, the anchored confidence rubric, the quote-the-line gate, and the false-positive catalog. Do not reconstruct that contract from this file; the details that make findings precise live only there.

Dispatch all 3 in parallel using the harness's agent/task tool. Omit the `mode` parameter so the user's configured permission settings apply.

**Persona agents (`correctness`, `quality`):** fill the subagent template with

1. the persona file's content,
2. the shared scope rules from `references/diff-scope.md`,
3. the JSON schema from `references/findings-schema.json`,
4. review context: the intent summary, file list, and diff.

Each returns one complete JSON payload — all schema fields, including `why_it_matters` and the full `evidence` array — directly to the orchestrator. This skill keeps no run directory, so the return is the only copy: a reviewer that omits those fields leaves the report unable to explain its own findings.

Persona subagents are **read-only**. They review and return JSON; they do not edit project files. Non-mutating inspection (`git diff`, `git show`, `git blame`, `git log`) is permitted.

**Learnings researcher:** dispatch with its persona file, the same review context, and the resolved `<root>`. Its output is unstructured markdown that Stage 5 folds into the action items; it does not participate in the merge and never gets a section of its own.

**Sequential fallback:** if the harness cannot dispatch in parallel, spawn in the order correctness, quality, learnings-researcher. Never insert shell no-ops, sleeps, status polls, or "still waiting" turns to await subagents — they return on their own tool call.

### Stage 4: Merge findings

Process the 2 JSON returns (correctness + quality) into one deduplicated finding set.

1. **Validate.** Check each return for the required top-level fields (`reviewer`, `findings`, `residual_risks`, `testing_gaps`) and each finding for its required fields and legal enum values. Drop malformed returns or findings. A dropped *reviewer* means one of the two lenses did not run — surface that as a Notes line, never silently absorb it.
2. **Confidence gate.** Drop findings below anchor 50. At anchor 50, a finding surfaces only when it is P0 or when a concrete action can still be named for it. Anchors 75 and 100 always survive. The suppressed count is not reported.
3. **Quote-the-line gate.** A finding claiming anchor 75 or 100 whose first evidence item is not a verbatim `file:line` quote is demoted to anchor 50, then re-gated by step 2.
4. **Action gate.** Every surviving finding must name one concrete action — the `suggested_fix`, or an action derivable from `why_it_matters` plus the cited code. A finding that reduces to "consider" or "might want to" with no nameable change is dropped here, not softened into an advisory line.
5. **Deduplicate.** Fingerprint each finding as `normalize(file) + line_bucket(line, +/-3) + normalize(title)`. On a match, merge: keep the highest severity, keep the highest anchor, and list both reviewers on the item.
6. **Separate pre-existing.** Pull findings with `pre_existing: true` into their own list. They are one line each and are never counted in the blocking total.
7. **Promote testing gaps.** A `testing_gaps` entry that is not already covered by a finding and that names a test worth writing becomes its own action item (`Add a test for <behavior>`), severity P2 unless the untested path is a P0/P1 failure mode. Discard the rest. Do not emit a testing-gaps inventory.
8. **Sort and number.** Sort by fix order: severity (P0 first), then anchor (descending), then file path, then line number. Number the result `1..N` in that fix order, so `#1` is the most urgent item. Do not reorder for prerequisites — state a dependency in the item's body instead. **The report then prints this list in reverse**, so `#1` is the last item on screen (Stage 5).

### Stage 5: Synthesize and present

Read `references/review-output-template.md` from this skill's directory and assemble the report to its structure. The output is an **action list** — the reader will either apply these items to their own change or paste them as review feedback on someone else's.

**The report is read bottom-up, so it is written in reverse order of importance.** When output ends the terminal viewport sits on the last line, so the last thing printed must be the thing to do first. Context goes at the top, where it is scrolled past; the action items go last. Render in this order:

1. **Header.** Action-item count and blocking count, scope, intent.
2. **Notes.** At most 3 lines, and only per the template's Notes rule.
3. **Not from this change.** Pre-existing findings, one line each.
4. **Action items, bucketed by ascending urgency.** `Optional` (P3), then `Worth fixing` (P2), then `Fix before merge` (P0/P1) as the **final section of the report**. Each item is an imperative title, a `file:line`, a `(severity, reviewer, confidence)` tag, and one to three sentences: what breaks, then the concrete fix. Within a bucket, order by confidence anchor **ascending**, so the last item printed is the highest-confidence blocker. Omit empty buckets. Nothing informational follows the last item.

**Fold the learnings researcher's output into the items** — it gets no section of its own. A past learning that applies to a specific item becomes a Notes line referencing that item's number; a learning that identifies a defect the personas missed becomes its own action item, subject to the same action gate; a learning with no action attached is dropped.

**Before delivering, verify** the report contains no blockquote, no horizontal rule, no Coverage section, no Learnings section, and no verdict line, and that its **last line belongs to the highest-priority action item** (or, with no items at all, to the empty-state line), with every informational section above the buckets. If the diff warrants reviewers this roster does not have (auth, migrations, public API contracts, CI/deploy gates), that is a single Notes line recommending `/ce-code-review` — use `$ce-code-review` when the active harness is Codex or otherwise documents dollar-prefixed invocation. Output exactly one form.

### Headless output format

In `mode:headless`, replace the interactive report with this envelope. Same action list, same numbering, plus the routing fields a downstream resolver needs. This envelope is **parsed, not scrolled**, so it stays in ascending fix order (`[1]` first) — the reverse-order rule applies only to the interactive report:

```
Quick review complete (headless mode).

Scope: <scope-line>
Intent: <intent-summary>
Actions: <N> (<M> blocking)

[1][P0][gated_auto -> downstream-resolver] <file:line> -- <imperative title> (correctness, confidence 100)
  Why: <why_it_matters>
  Fix: <suggested_fix>
  Evidence: <first evidence item>

[2][P2][manual -> human] <file:line> -- <imperative title> (quality, confidence 75)
  Why: <why_it_matters>
  Fix: <suggested_fix>

Pre-existing:
[P2] <file:line> -- <title> (correctness)

Notes:
- <note>

Review complete
```

`Actions` counts the numbered items; `<M> blocking` counts P0 and P1. Omit any section with zero items. End with `Review complete` as the terminal signal.

## Quality Gates

Before delivering the report, verify:

1. **Every item names an action.** Each item must survive being read as an instruction to the change's author. "Consider" or "might want to" with no concrete change is not an item — name the change or drop it.
2. **No false positives from skimming.** Each finding's evidence must show the surrounding code was actually read, not pattern-matched.
3. **Severity is calibrated.** A style nit is never P0. A data-loss bug is never P3. Severity decides the bucket, so a miscalibrated severity puts a blocker under `Optional`.
4. **Line numbers are accurate.** Verify each cited line against the file content; a wrong `file:line` makes a correct item unusable as review feedback.
5. **Nothing found is stated as such, not as a clearance.** This roster has three reviewers. An empty list means "these three found nothing," not "this change is safe."
