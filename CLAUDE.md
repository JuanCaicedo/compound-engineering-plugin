@AGENTS.md

# Juan Caicedo's Fork

> **Fork Information:** This is a private fork of [EveryInc/compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin). Rails, Ruby, and Python components have been removed, and GitLab/Vanna-specific skills have been added. See [FORK.md](FORK.md) for the full customization list and the upstream sync workflow.

`AGENTS.md` (imported above) is the canonical repository instruction file — repo structure, working agreements, conventions, and plugin-maintenance rules all live there and apply to this fork unchanged. This file only records fork-specific context.

## Fork-specific notes

- **Layout:** Upstream is now root-native. Skills live in top-level `skills/`; the plugin ships no standalone `agents/` or `commands/`. Reviewer/research personas are frontmatter-free prompt assets under `skills/<skill>/references/{personas,agents}/`.
- **Removed:** Rails/Ruby/Python reviewer agents and skills (see FORK.md for the exact list). When syncing, keep these deletions and strip any re-introduced references.
- **Added:** `ce-agent-review`, `ce-resolve-mr-feedback` (GitLab MR resolution via `glab`), `ce-quick-review`, `ce-generate-review-guide`, `ce-vanna-patterns-review`, and the `brandon-aldrich-reviewer` / `jeremy-gillick-reviewer` / `quality-reviewer` personas.
- **Behavioral tweaks:** `ce-code-review` treats `agent-native-reviewer` and `learnings-researcher` as conditionals (not always-on) and adds `tier:lean`; `ce-plan` / `ce-brainstorm` use `ft-<ticket>` plan filenames instead of the upstream `NNN` sequence.
- **Validation:** run `bun install && bun run release:validate && bun test` after any sync or component change. Counts are release-owned — do not hand-edit them.
- **Git:** do not auto-commit; wait for explicit instruction. Point repository URLs at `JuanCaicedo/compound-engineering-plugin`.
