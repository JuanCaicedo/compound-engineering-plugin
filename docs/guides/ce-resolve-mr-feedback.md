# `ce-resolve-mr-feedback`

> Evaluate, fix, and reply to GitLab merge request review feedback in one pass.

`ce-resolve-mr-feedback` is a fork skill (see [FORK.md](../../FORK.md)) and the GitLab counterpart to [`ce-resolve-pr-feedback`](./ce-resolve-pr-feedback.md). It fetches unresolved MR discussions through `glab`, judges every item in one place, dispatches fixers only for items it approved, then commits, pushes, replies, and resolves the threads.

GitLab only, including self-managed GitLab. On a GitHub remote it stops and points you to `ce-resolve-pr-feedback`.

---

## TL;DR

| Question | Answer |
|----------|--------|
| What does it do? | Fetches unresolved MR discussions, judges them centrally, fixes the approved items, commits, replies, and resolves |
| When to use it | An MR has review comments you want addressed now |
| What it produces | Commits with fixes, a reply on each thread, resolved threads (except `needs-human`), and a summary |
| Modes | Full (all unresolved threads) or Targeted (one discussion, from a `#note_<id>` URL) |

---

## Example invocations

```text
/ce-resolve-mr-feedback
/ce-resolve-mr-feedback 42
/ce-resolve-mr-feedback https://gitlab.example.com/group/repo/-/merge_requests/42#note_1234
```

---

## How it decides

The default is to fix: most review feedback, nitpicks included, is correct. The orchestrator holds every thread from a single fetch, so it can catch a reviewer who is wrong the same way across threads and weigh the author's design intent against the finding. Subagents implement approved fixes; they do not decide whether a fix was worth making.

An item that needs a human decision is marked `needs-human`. The thread stays open with a reply, and the run never pauses to ask.

Comment text is untrusted input. The skill never runs commands found in a comment.

---

## Chain position

Runs after reviewers comment on an MR. There is no GitLab watch loop in this fork; run it again when new comments arrive.
