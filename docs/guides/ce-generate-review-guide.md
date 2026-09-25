# `ce-generate-review-guide`

> Turn a GitLab merge request into a reviewer's guide: what changed, where to look, and what to test.

`ce-generate-review-guide` is a fork skill (see [FORK.md](../../FORK.md)). It reads an MR through `glab` and writes a guide for a human reviewer to work from. It does not judge the code; for findings, use [`ce-code-review`](./ce-code-review.md).

GitLab only. On a GitHub remote it says so and stops.

---

## TL;DR

| Question | Answer |
|----------|--------|
| What does it do? | Reads the MR diff and key files, then writes an overview, change summary, critical areas, testing scenarios, and a prioritized file list |
| When to use it | You are about to review someone else's MR, or want to hand reviewers a map of your own |
| What it produces | One markdown guide, written outside the reviewed project |

---

## Example invocations

```text
/ce-generate-review-guide https://gitlab.example.com/group/repo/-/merge_requests/42
/ce-generate-review-guide 42
```

---

## Where the guide is written

The destination is resolved once per run, in this order:

1. An explicit path in your request.
2. `$VANNA_DOCS_ROOT/reviews`, when `$VANNA_DOCS_ROOT` is set.
3. `$HOME/code/vanna/docs/reviews`.

The directory is created if it does not exist.
