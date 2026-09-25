# `ce-quick-review`

> A fast pre-PR review with a fixed three-agent team. One numbered action list, no fixes applied.

`ce-quick-review` is a fork skill (see [FORK.md](../../FORK.md)). It is the lightweight tier below [`ce-code-review`](./ce-code-review.md): every run dispatches the same three reviewers in parallel, with no diff-aware persona selection, no PR target, no plan verification, and no local apply.

---

## TL;DR

| Question | Answer |
|----------|--------|
| What does it do? | Runs `correctness`, a combined `quality` reviewer (maintainability plus testing), and a learnings researcher, then merges their findings into one list |
| When to use it | Fast feedback on a small or medium local change before opening a PR |
| What it produces | A bottom-up action list: context at the top, then `Optional` -> `Worth fixing` -> `Fix before merge`, so the report ends on the most urgent item |
| Modes | Interactive report (default) and `mode:headless`, a parsed envelope in ascending fix order |

---

## Example invocations

```text
/ce-quick-review
/ce-quick-review base:origin/main
/ce-quick-review mode:headless
```

---

## The reviewers

| Reviewer | Focus | Model |
|----------|-------|-------|
| `correctness` | Logic errors, edge cases, state bugs, error propagation, intent mismatches | Session model |
| `quality` | Naming, abstraction, dead code, coverage gaps, brittle tests | Mid-tier |
| `learnings-researcher` | Past learnings under `docs/solutions/` that apply to this diff | Mid-tier |

Learnings do not get their own section. A learning that applies to an item becomes a Notes line naming that item; a learning that finds a defect the reviewers missed becomes its own action item.

---

## When to use `ce-code-review` instead

Use [`ce-code-review`](./ce-code-review.md) when the diff touches auth, data migrations, public API contracts, or a CI, deploy, coverage, or lint gate. Also use it when you need a PR target, plan verification, or local apply. This roster has no reviewer for those concerns and will not tell you one is missing.

---

## Chain position

Sits after [`ce-work`](./ce-work.md) and before [`ce-commit-push-pr`](./ce-commit-push-pr.md) for changes that do not warrant the full review.
