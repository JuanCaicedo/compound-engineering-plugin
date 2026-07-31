# Fork Management

This is Juan Caicedo's fork of [EveryInc/compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin).

## What This Fork Adds

The fork exists to carry **GitLab** and **Vanna**-specific skills that upstream does not want, since upstream is GitHub-only by design.

| Skill | Purpose |
|-------|---------|
| `ce-resolve-mr-feedback` | GitLab counterpart to upstream's `ce-resolve-pr-feedback`. Speaks GitLab discussions via `glab`. |
| `ce-generate-review-guide` | Produces a reviewer's guide from a GitLab MR URL. |
| `ce-vanna-patterns-review` | Reviews a branch against the patterns documented in the Vanna docs repository. |

One upstream skill is modified:

- **`ce-plan`** — plan filenames use a ticket identifier (`YYYY-MM-DD-ft-<ticket>-<type>-<name>-plan.md`) instead of upstream's daily sequence number (`-NNN-`).

Everything else tracks upstream unchanged.

## What This Fork No Longer Does

Earlier versions of this fork removed Rails, Ruby, and Python components (`dhh-rails-reviewer`, `kieran-rails-reviewer`, `kieran-python-reviewer`, `dhh-rails-style`, `andrew-kane-gem-writer`, `dspy-ruby`). **Upstream has since removed all of them itself**, so those deletions are no longer fork customizations and require no maintenance.

The fork also previously carried its own reviewer personas (`brandon-aldrich-reviewer`, `jeremy-gillick-reviewer`, a combined `quality-reviewer`) and an `agent_review` command. These were dropped during the July 2026 sync — upstream's persona set and `ce-code-review` dispatch supersede them.

## Upstream Layout Change (July 2026)

Upstream moved the plugin from `plugins/compound-engineering/**` to the **repository root**. This is the single most important thing to know when reading old fork history:

| Before | After |
|--------|-------|
| `plugins/compound-engineering/skills/<name>/` | `skills/<name>/` |
| `plugins/compound-engineering/agents/<category>/<name>.md` | `skills/<skill>/references/agents/<name>.md` |
| `plugins/compound-engineering/commands/<name>.md` | *(removed — commands no longer exist)* |
| `plugins/compound-engineering/.claude-plugin/plugin.json` | `plugin.json` and `.claude-plugin/plugin.json` |

Consequences for fork maintenance:

- **No standalone agents.** Specialist personas live inside the skill that dispatches them, as frontmatter-free prompt assets under `references/agents/` or `references/personas/`. See AGENTS.md "Specialist Prompt Assets in Skills".
- **No commands.** Upstream removed the top-level command surface ("agentless plugin surface reduction"). Anything that was a command is now a skill.
- **Root `CLAUDE.md` is a symlink to `AGENTS.md`.** Do not replace it with a regular file — `claude plugin validate --strict` fails when the plugin root has a real `CLAUDE.md`.
- **The `coding-tutor` plugin is gone.** Upstream deprecated and removed it; the fork never customized it.

## Conventions Fork Skills Must Follow

Upstream enforces these with tests. A new fork skill that ignores them will fail `bun run test`.

- **`ce-` name prefix.** Skill directory and frontmatter `name` must start with `ce-` (`tests/skill-agent-ce-prefix.test.ts`). Upstream keeps an exemption allowlist in that test — **do not add fork skills to it**, because editing an upstream-owned test file creates a conflict on every future sync. Prefix instead.
- **Self-contained references.** A skill may only reference files inside its own directory. No `../other-skill/...`, no absolute paths into the plugin. Duplicate a shared file rather than reaching for it.
- **No `!`cmd`` pre-resolution** in SKILL.md. Gather context at runtime as single argv-style commands whose exit status is read as control flow.
- **Prompt assets carry no YAML frontmatter.** Model and tool policy belong in the calling SKILL.md.
- **LF line endings** on bundled scripts (`tests/bundled-script-line-endings.test.ts`).

Fork skills that read a docs repository outside the current project (`ce-generate-review-guide`, `ce-vanna-patterns-review`) resolve it as `$VANNA_DOCS_ROOT` first, then `$HOME/code/vanna/docs`, and fail loudly rather than silently reviewing against nothing.

## Syncing With Upstream

```bash
# 1. Fetch and see what's new
git fetch upstream
git log HEAD..upstream/main --oneline

# 2. Sync on a branch
git checkout -b sync-upstream-$(date +%Y%m%d)
git merge upstream/main

# 3. Resolve conflicts (see below), then validate
bun install
bun run test
bun run release:validate

# 4. Land it
git checkout main
git merge sync-upstream-$(date +%Y%m%d)
git push origin main
```

### Conflict Resolution

Take upstream's side by default. The fork's surface is small and deliberate; anything outside it should track upstream exactly.

| Conflict | Resolution |
|----------|-----------|
| `.claude-plugin/marketplace.json` | Keep fork `name`/`owner`/`homepage` and the fork description. Take upstream's `metadata.version` and plugin `source` — those are release-owned. |
| `README.md` | Take upstream's content, then re-point install paths to `JuanCaicedo/...` and keep the fork notice near the top. |
| `AGENTS.md` / `CLAUDE.md` | Take upstream. `CLAUDE.md` must stay a symlink to `AGENTS.md`. |
| `skills/ce-plan/SKILL.md` | Take upstream, then re-apply the `ft-<ticket>` naming (two spots: Phase 3.1 file naming, and the save-path block). |
| A fork skill | Fork-owned; keep the fork's version. |
| Anything else | Take upstream. |

### Versions

Do not hand-bump versions in `plugin.json` or `marketplace.json` — upstream's release automation owns them, and hand edits cause version drift. Let the merge bring whatever upstream set.

### After Syncing

```bash
bun run test                 # skill-convention and contract guards
bun run release:validate     # plugin/marketplace consistency
jq . .claude-plugin/marketplace.json
jq . .claude-plugin/plugin.json
```

Then confirm the fork surface survived:

```bash
ls skills/ | grep -E 'ce-(resolve-mr-feedback|generate-review-guide|vanna-patterns-review)'
grep -c 'ft-<ticket>' skills/ce-plan/SKILL.md   # expect 2 or more
```

## Contact

- **Fork maintainer**: Juan Caicedo (@JuanCaicedo)
- **Upstream project**: [EveryInc/compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin)
- **Upstream maintainers**: Kieran Klaassen (@kieranklaassen), Trevin Chow
