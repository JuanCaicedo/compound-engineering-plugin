# Fork Management

This is Juan Caicedo's fork of [EveryInc/compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin).

## What This Fork Adds

The fork exists mainly to carry **GitLab** skills, which upstream does not want — upstream is GitHub-only by design.

| Skill | Purpose |
|-------|---------|
| `ce-resolve-mr-feedback` | GitLab counterpart to upstream's `ce-resolve-pr-feedback`. Speaks GitLab discussions via `glab`. |
| `ce-generate-review-guide` | Produces a reviewer's guide from a GitLab MR URL. |
| `ce-quick-review` | Lightweight review tier: a fixed 3-agent roster (correctness, combined quality, learnings) with no conditional persona selection, PR targeting, or apply path. Upstream folds the "quick review" ask into `ce-code-review`'s short-circuit, which delegates to the harness's built-in review instead of running CE personas. |

Upstream skills are modified in two places:

- **`ce-plan`** — plan filenames use a ticket identifier (`YYYY-MM-DD-ft-<ticket>-<type>-<name>-plan.md`) instead of upstream's wall-clock prefix (`-HHMM-`, formerly a daily `-NNN-` sequence). Open Questions must also survive a resolution pass (code, origin artifacts, obvious default) before they stay in a plan.
- **Review report shape** — `ce-code-review`'s markdown report and `ce-quick-review`'s report are a bottom-up action list: context first, then `Optional` -> `Worth fixing` -> `Fix before merge`, so the report ends on the most urgent item. There is no "Actionable Findings" section; consumers (`ce-work`, `lfg`) read `actionable_findings` from `mode:agent` JSON or the markdown items routed `-> downstream-resolver`.
- **Review rosters trimmed** — `ce-code-review` drops the `swift-ios` reviewer (no iOS work) and `previous-comments` (it only fires on GitHub PR targets, and this fork reviews GitLab MRs). Its `performance` reviewer and `ce-simplify-code`'s efficiency reviewer run only when the change adds cost that grows with data or traffic; most reviews skip them.
- **Cross-model peer review is disabled fork-wide** — see below.

Everything else tracks upstream unchanged.

## No Content Leaves the Machine

Upstream ships four surfaces that send project content to an external model CLI (Codex, Claude, Grok, Cursor/Composer). **All four are disabled in this fork.** No code, diff, document, plan, prompt brief, or unit packet is sent to a peer model for review, judgment, or implementation. (One unrelated opt-in path remains — see "Not covered" below.)

| Surface | Upstream behavior | Fork behavior |
|---|---|---|
| `ce-code-review` | Cross-model adversarial pass ships the working-tree diff to a peer CLI | Never starts a peer. The in-process `adversarial-reviewer` always owns the lens, in every scope mode. |
| `ce-doc-review` | Cross-model judgment pass ships the document and per-lens slices | Never runs. Every lens is satisfied in-process. |
| `ce-pov` | Cross-model panel ships the subject and consults peers | Never convenes. A peer/`oracle` summons returns the solo POV plus an explicit disabled note. |
| `ce-work` | Cross-model execution engine ships bounded units to an external harness to author | Not selectable. Always implements natively. `implementation_run:` recovery is also disabled, since resuming reopens the channel. |

**The machinery stays on disk.** `cross-model-*.md` references, `cross-model-*.sh` workers, and `peer-job-runner.py` are untouched so upstream merges apply cleanly and the parity tests keep passing. Each dormant reference carries a `DISABLED IN THIS FORK` banner. The gates are in the orchestration prose, at the point where each skill would otherwise resolve a route or start a job — that is the layer to re-check after every sync.

The peer and external-worker config keys are inert and documented as such in `config-template.yaml`, its byte-identical `.compound-engineering/config.example.yaml` copy, and `docs/guides/configuration.md`: `cross_model_review_mode`, `cross_model_peer`, `cross_model_model`, `cross_model_effort`, `work_engine_mode`, `work_engine_preferences`, and `work_engine_effort`. Setting them, including `cross_model_review_mode: auto`, does not re-enable anything.

**Not covered: model elevation.** `ce-plan` and `ce-brainstorm` share the same `peer-job-runner.py` plumbing for a *different* feature — dispatching one reasoning-heavy step to a user-chosen model (`references/reasoning-elevation.md`). It is **not** cross-model peer review: it never routes to Codex, Cursor, or Grok, its only off-host adapter is the `claude` CLI, and it is off unless `plan_model` / `brainstorm_model` is set or the prompt explicitly asks for it. It was left alone because it is out of scope for "stop shipping code to Codex/Cursor" — but it *can* invoke an external CLI, so disable it too by leaving those config keys unset if you want a strictly no-egress checkout.

**Re-enabling** means reverting the prose gates in the files listed in the conflict table below — there is no runtime switch, deliberately. Upstream's own `CROSS_MODEL_MAX_PEERS=0` env var also stops `ce-code-review` and `ce-doc-review` at the worker (before any egress), but it does not cover `ce-pov` or `ce-work`, which is why the fork gates in prose instead.

## What This Fork No Longer Does

Earlier versions of this fork removed Rails, Ruby, and Python components (`dhh-rails-reviewer`, `kieran-rails-reviewer`, `kieran-python-reviewer`, `dhh-rails-style`, `andrew-kane-gem-writer`, `dspy-ruby`). **Upstream has since removed all of them itself**, so those deletions are no longer fork customizations and require no maintenance.

The fork also previously carried its own reviewer personas (`brandon-aldrich-reviewer`, `jeremy-gillick-reviewer`, a combined `quality-reviewer`) and an `agent_review` command. `brandon-aldrich-reviewer`, `jeremy-gillick-reviewer`, and `agent_review` were dropped during the July 2026 sync — upstream's persona set and `ce-code-review` dispatch supersede them.

The combined `quality-reviewer` is the exception: it came back as a skill-local prompt asset at `skills/ce-quick-review/references/personas/quality-reviewer.md`, since `ce-quick-review` is the reason that persona exists (one subagent covering maintainability *and* testing, instead of upstream's two). Its confidence calibration was rewritten for upstream's anchored 0/25/50/75/100 rubric; the old 0.0-1.0 float scale is gone from the codebase.

### The `ce-quick-review` loss and restore

`quick-review` was a fork skill at `plugins/compound-engineering/skills/quick-review/` that the July 2026 layout sync silently dropped — the merge renamed `plugins/compound-engineering/skills/` to `skills/`, and a fork skill with no upstream counterpart did not survive the rename. Because the loss happened inside the merge, `git log -- '*quick-review*'` shows no deletion commit; the skill is still present on the pre-sync `main`.

It was restored onto the root-native layout rather than reverted verbatim: upstream's review substrate had moved (anchored integer confidence, `autofix_class` without `safe_auto`, `owner` without `review-fixer`, personas that defer to the subagent template's rubric), so a verbatim restore would have shipped a skill contradicting its own schema. The restored skill keeps the original's shape — fixed 3-agent roster, report-only, single merged report — on current parts.

**Watch for this failure mode on the next sync.** Any fork skill can vanish the same way if upstream moves paths again. After a sync, verify the fork surface with the commands under "After Syncing" below rather than assuming a clean merge preserved it.

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

`ce-generate-review-guide` writes outside the current project, into a central docs repository. It resolves that location as `$VANNA_DOCS_ROOT` first, then `$HOME/code/vanna/docs`.

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
| `skills/ce-plan/**` | Take upstream, then re-apply the `ft-<ticket>` naming (two spots: the filename step in `references/structure.md`, and the save-path block in `references/final-review.md`) and the Open Questions checklist line in `references/final-review.md`. `SKILL.md` itself is now a thin upstream spine with no fork edit. |
| `tests/release-metadata.test.ts` | Asserts a hardcoded skill count. Take upstream's number and add the fork's skill count on top — do not revert to the upstream literal. |
| Any cross-model gate (see table below) | Take upstream's content, then re-apply the fork's disable gate. Never accept an upstream hunk that restores a peer dispatch, route resolution, or egress announcement. |
| A fork skill | Fork-owned; keep the fork's version. |
| Anything else | Take upstream. |

#### Upstream-owned files carrying a fork edit

Expect each of these to conflict whenever upstream touches it. Every other fork change lives in fork-owned files.

| File | Fork edit |
|---|---|
| `.gitattributes` | `eol=lf` pin for the fork's extensionless scripts |
| `tests/release-metadata.test.ts` | Skill count (upstream's number plus the fork's 3) |
| `skills/ce-code-review/references/{persona-catalog,select-and-route,scope}.md`, `scripts/review-scope.py`, `skills/ce-simplify-code/SKILL.md`, `docs/guides/{ce-code-review,ce-simplify-code}.md` | Trimmed roster; performance and efficiency skip by default. If upstream edits a removed persona file, keep it deleted |
| `tests/codex-skill-prompt-budget.test.ts` | `ce-quick-review` listed in `OVER_BUDGET` until it is split under Codex's 8000-byte cap |
| `README.md` | Fork badge, install paths, skill counts, and a `Fork additions` row in "Skills at a glance" (the metadata test requires every skill to be named there) |
| `skills/ce-plan/references/{structure,final-review,plan-sections}.md` | `ft-<ticket>` naming, Open Questions resolution pass |
| `tests/skills/unified-plan-artifact-contract.test.ts` | Plan-filename test rewritten to pin `ft-<ticket>` |
| `skills/ce-code-review/references/{review-output-template,finish-review}.md`, `docs/guides/ce-code-review.md` | Bottom-up action-list report |
| `skills/ce-work/references/{review-findings-followup,shipping-workflow}.md`, `skills/lfg/**` | Read the actionable set instead of the removed Actionable Findings section |
| `tests/review-skill-contract.test.ts`, `tests/fixtures/ce-code-review-stable-numbering.md` | Pin the action-list contract |
| `skills/ce-code-review/SKILL.md` | Cross-model disable gate (spine step 5, Stage 3d); names `cross_model_review_mode` / `cross_model_peer` as inert |
| `skills/ce-code-review/references/{select-and-route,depth-paths,finish-input,modes-and-output}.md` | Stage 3d route is always local; the focused depth path's independent read is a local `adversarial-reviewer`, not a peer; `peer` fields fixed to not-run |
| `skills/ce-code-review/references/{dispatch-reviewers,finish-review,persona-catalog}.md` | Peer fold-in, promotion, and Coverage wording reconciled to in-process only |
| `skills/ce-doc-review/SKILL.md` | Cross-model judgment pass disabled |
| `skills/ce-doc-review/references/synthesis-and-presentation.md` | Header note marking the peer rules inert |
| `skills/ce-pov/SKILL.md` | Panel disabled; description and `argument-hint` reconciled |
| `skills/ce-pov/references/invocation.md` | `oracle` / named-peer wording reconciled |
| `skills/ce-work/SKILL.md` | Cross-model engine and `implementation_run:` recovery disabled |
| `skills/ce-work/references/{execution-engines,implementation-loop}.md` | Engine removed from selection |
| `skills/*/references/cross-model-*.md` | `DISABLED IN THIS FORK` banner |
| `skills/ce-setup/references/config-template.yaml` + `.compound-engineering/config.example.yaml` | `cross_model_*` and `work_engine_*` keys marked inert (keep the two byte-identical) |
| `docs/guides/configuration.md` | Same two rows marked inert |
| `tests/pov-skill-contract.test.ts` | 4 panel tests rewritten to pin the disabled contract |
| `tests/skills/task-visibility-contract.test.ts` | Peer-task test rewritten to pin no-egress |
| `tests/skills/ce-work-outcome-spine.test.ts` | 6 engine/recovery tests rewritten to pin native-only execution |

If an upstream sync adds a **new** cross-model surface, it arrives ungated — the gates above are prose, not a switch, so nothing stops a newly added path. Grep for it after every sync (see "After Syncing").

### Eval refs that live only on upstream branches

`tests/skill-eval-cell/catalog.ts` pins commit SHAs, and upstream sometimes pins one that exists only on an unmerged upstream branch. The fork's CI cannot see those, so `catalog.test.ts` fails with `<skill> missing at <sha>`. After a sync, find any such ref and push it to the fork as a tag (CI checks out with `fetch-depth: 0`, which includes tags):

```bash
for r in $(grep -ohE '"[0-9a-f]{40}"' tests/skill-eval-cell/*.ts | tr -d '"' | sort -u); do
  git merge-base --is-ancestor "$r" HEAD || echo "$r"
done
git tag upstream-pin/<short-sha> <sha> && git push origin upstream-pin/<short-sha>
```

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
ls skills/ | grep -cE 'ce-(resolve-mr-feedback|generate-review-guide|quick-review)'   # expect 3
grep -c 'ft-<ticket>' skills/ce-plan/SKILL.md   # expect 2 or more
grep -rlc 'disabled in this fork' skills/ce-code-review/SKILL.md skills/ce-doc-review/SKILL.md skills/ce-pov/SKILL.md skills/ce-work/SKILL.md   # expect all 4
```

A count below 3 means the sync dropped a fork skill — recover it from the pre-sync commit, not from upstream.

A missing disable gate means the sync reverted one; re-apply it before running anything that reviews code. Then check whether upstream added a **new** egress path the gates do not cover:

```bash
git diff HEAD@{1} --stat -- 'skills/**/cross-model*' 'skills/**/peer-job-runner.py'
grep -rln 'cross-model\|CROSS_MODEL' skills/ | cut -d/ -f2 | sort -u   # any skill not in the disabled table above is ungated
```

## Contact

- **Fork maintainer**: Juan Caicedo (@JuanCaicedo)
- **Upstream project**: [EveryInc/compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin)
- **Upstream maintainers**: Kieran Klaassen (@kieranklaassen), Trevin Chow
