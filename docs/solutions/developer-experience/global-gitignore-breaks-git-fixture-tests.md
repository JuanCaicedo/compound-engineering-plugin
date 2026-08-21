---
title: "A global gitignore pattern silently fails git-fixture tests that CI passes"
date: 2026-08-21
problem_type: developer_experience
category: developer-experience
component: test_suite
root_cause: environment_configuration
resolution_type: diagnosis_guidance
severity: medium
tags: [testing, git, gitignore, false-failure, fixtures, ci-parity]
applies_when:
  - "A local `bun run test` fails tests that pass in CI"
  - "Deciding whether a local-only test failure is a real defect or an environment artifact"
  - "Test fixtures build throwaway git repos with `git init` and `git add`"
---

# A global gitignore pattern fails git-fixture tests that CI passes

## Context

On a local checkout, `bun run test` reported 13 failures across three files while the same
commit's CI run reported a single, unrelated failure:

```
(fail) ce-code-review deterministic mechanics > scope helper counts executable changes and fails closed on uncounted files
  expect(scope.uncounted_files).toBe(1)   // Received: 0
(fail) ce-work unit workspace controller > retries an abandoned unit under the same run ...
  expect(initWithBinding(...).word).toBe("READY")   // Received: "REFUSED"
```

The cause is not in the repo. Test fixtures build throwaway repositories with `git init`,
and a fresh repository still reads the developer's **global** git config — including
`core.excludesFile`. A global ignore entry as ordinary as a bare `docs` line makes
`git add .` silently skip every fixture file written under `docs/`, so:

- `review-scope.py` never sees `docs/note.md` in the diff, and `uncounted_files` is 0 rather than 1.
- `unit-workspace.py` cannot find the fixture plan at `docs/plans/plan.md`, and `init` answers `REFUSED`.

CI passes because a GitHub runner has no global git config to inherit. Nothing about the
failing assertions is wrong, and nothing in the scripts they exercise needs changing.

## Diagnosis

Re-run the suite with the global config neutralized. If the failures disappear, they are
environmental:

```bash
GIT_CONFIG_GLOBAL=/dev/null bun run test
```

Then confirm the pattern that did it:

```bash
git config --global --get core.excludesFile
grep -nE 'docs|\*\.md' "$(git config --global --get core.excludesFile)"
```

A bare directory name in a global ignore file (`docs`, `build`, `tmp`, `notes`) matches that
name at **every** depth in **every** repository, including temp-directory fixtures. The
narrower fix is to scope such patterns per-repo, or anchor them, rather than carrying a bare
name globally.

## Guidance

- **Prove CI parity before editing source.** A local-only failure in a git-fixture test is an
  environment claim until `GIT_CONFIG_GLOBAL=/dev/null` says otherwise. Do not "fix" a script
  whose assertion is correct on a clean machine.
- **Do not paper over it with a repo-wide preload.** Adding a root `bunfig.toml` that presets
  `GIT_CONFIG_GLOBAL` for the whole suite was measured on this repo and made things worse —
  117 failures instead of 13, across ce-work suites that pass individually. The suite is
  sensitive to root-level configuration in ways that are not worth discovering under time
  pressure; keep the neutralization on the command line.
- **A bare `docs` ignore is dangerous beyond tests.** Any workflow that writes documentation
  into a `docs/` directory will see new files silently unstaged, with no error and no output
  from `git add`.
