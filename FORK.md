# Fork Management

This is Juan Caicedo's private fork of [EveryInc/compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin).

## Customizations

This fork includes the following customizations:
- Removed Rails/Ruby-specific components (agents and skills)
- Removed Python-specific components (kieran-python-reviewer agent)
- Fork-specific skills added (top-level `skills/`, all `ce-` prefixed):
  - `ce-vanna-patterns-review`, `ce-generate-review-guide` — GitLab/Vanna workflow helpers
  - `ce-agent-review` — single-persona focused review (converted from the old `/agent_review` command)
  - `ce-resolve-mr-feedback` — GitLab MR review-thread resolution via `glab` (the GitLab counterpart to upstream's `ce-resolve-pr-feedback`; converted from `/resolve_mr_parallel`)
  - `ce-quick-review` — lightweight 3-agent review
  - Fork reviewer personas under `skills/ce-code-review/references/personas/`: `brandon-aldrich-reviewer`, `jeremy-gillick-reviewer`, `quality-reviewer`
- `ce-code-review`: `agent-native-reviewer` and `learnings-researcher` moved from the always-on set to CE conditionals; added a `tier:lean` argument (inverse of `depth:full`)
- `ce-plan` / `ce-brainstorm`: plan filenames use `ft-<ticket>` (e.g. `2026-01-21-ft-123-feat-...`) instead of the upstream `NNN` daily sequence number

> **Layout note (post-3.15.0 sync):** Upstream moved the plugin to a root-native layout. Skills now live in top-level `skills/`; the plugin no longer ships standalone `agents/` or `commands/` (reviewer/research agents are frontmatter-free prompt assets under `skills/<skill>/references/{personas,agents}/`). The old `plugins/compound-engineering/` tree no longer exists. Current counts: 0 standalone agents, 31 skills, 0 MCP servers.

## Syncing with Upstream

### Setup (already done)

```bash
# Add upstream remote (already configured)
git remote add upstream https://github.com/EveryInc/compound-engineering-plugin.git
```

### Regular Sync Workflow

```bash
# 1. Fetch latest changes from upstream
git fetch upstream

# 2. Check what's changed
git log HEAD..upstream/main --oneline

# 3. Create a sync branch
git checkout -b sync-upstream-$(date +%Y%m%d)

# 4. Merge upstream changes
git merge upstream/main

# 5. Resolve conflicts (see below)
# ... fix conflicts ...

# 6. Test the changes
git status
# Verify plugin still works

# 7. Merge to main
git checkout main
git merge sync-upstream-$(date +%Y%m%d)
git push origin main
```

## Handling Merge Conflicts

### CHANGELOG.md Conflicts (Common)

The CHANGELOG will always conflict because both upstream and your fork add entries at the top.

**Resolution strategy:**
1. Keep BOTH changelog entries
2. Order them by version number (higher versions first)
3. Preserve your custom entries (2.29.0, 2.30.0)
4. Add upstream entries below

```markdown
# Changelog

...

## [2.30.0] - 2026-02-03 (Fork)
### Removed
- Python-specific components

## [2.29.0] - 2026-02-03 (Fork)
### Removed
- Rails/Ruby-specific components

## [2.28.0] - 2026-01-21 (Upstream)
### Added
- New features from upstream
```

**Tip:** Mark your fork-specific entries with `(Fork)` to distinguish them.

### Component Count Conflicts

Counts are release-owned now — do not hand-edit them. `bun run release:validate` recomputes counts from the repo and checks that release metadata is in sync:

```bash
bun run release:validate
```

Note: skills live in top-level `skills/`; the plugin ships no standalone `agents/` or `commands/`. Reviewer/research personas are prompt assets under `skills/<skill>/references/{personas,agents}/` and are not counted as agents.

### Deleted Component References

If upstream references components you've deleted (Rails/Python reviewers), remove those references during merge:

```bash
# Search for deleted component references
grep -rn "kieran-rails-reviewer\|dhh-rails-reviewer\|kieran-python-reviewer" skills/ .claude-plugin/

# Remove any found references manually
```

### File Deletion Conflicts

If upstream modifies a file you've deleted, Git will show a conflict. Choose to keep the deletion:

```bash
# For deleted Rails/Ruby/Python components
git rm path/to/deleted/component.md
```

## Testing After Sync

```bash
# 1. Install deps and validate release metadata (counts, descriptions)
bun install
bun run release:validate

# 2. Validate marketplace/plugin JSON
jq . .claude-plugin/marketplace.json
jq . .claude-plugin/plugin.json

# 3. Run the full test suite (conversion, writers, skill contracts)
bun test

# 4. Check for broken references to removed components
grep -rn "kieran-rails-reviewer\|dhh-rails-reviewer\|kieran-python-reviewer\|dhh-rails-style\|andrew-kane-gem-writer\|dspy-ruby" skills/

# 5. Test plugin installation from the fork
claude /plugin marketplace add /Users/juan.caicedo/code/personal/compound-engineering-plugin
claude /plugin install compound-engineering
```

## Upstream vs Fork Versions

**Upstream versions:** Follow the original project's semver
**Fork versions:** Extend with custom changes

| Version | Source | Description |
|---------|--------|-------------|
| 2.28.0 | Upstream | Last synced upstream version |
| 2.29.0 | Fork | Removed Rails/Ruby components |
| 2.30.0 | Fork | Removed Python components |
| 2.31.0+ | Mixed | Future upstream syncs + fork changes |

## Merge Conflict Resolution Checklist

When merging upstream changes:

- [ ] Fetch upstream: `git fetch upstream`
- [ ] Create sync branch
- [ ] Merge upstream/main
- [ ] Resolve CHANGELOG conflicts (keep both, mark fork entries)
- [ ] Update component counts in plugin.json, marketplace.json, README.md
- [ ] Remove references to deleted components
- [ ] Validate JSON files with `jq`
- [ ] Recount actual components
- [ ] Test plugin installation
- [ ] Commit with clear message: "Sync with upstream vX.Y.Z"
- [ ] Push to origin

## Preserving Fork Customizations

To ensure your customizations survive merges:

1. **Document deletions** - This FORK.md file lists removed components
2. **Use .gitattributes** - Mark files with merge strategies (optional)
3. **Review carefully** - Always review merge conflicts before committing
4. **Test thoroughly** - Verify counts and references after each sync

## When to Sync

- **Monthly**: Check for new features and bug fixes
- **Before major work**: Start with latest upstream code
- **After upstream releases**: Stay current with stable versions
- **Security updates**: Sync immediately for security patches

## Contact

- **Fork maintainer**: Juan Caicedo (@JuanCaicedo)
- **Upstream project**: [EveryInc/compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin)
- **Upstream maintainer**: Kieran Klaassen (@kieranklaassen)
