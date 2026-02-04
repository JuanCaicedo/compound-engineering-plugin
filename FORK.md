# Fork Management

This is Juan Caicedo's private fork of [EveryInc/compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin).

## Customizations

This fork includes the following customizations:
- Removed Rails/Ruby-specific components (agents and skills)
- Removed Python-specific components (kieran-python-reviewer agent)
- Streamlined to 25 agents, 24 commands, 12 skills

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

If upstream adds new agents/commands/skills, you'll need to update counts:

```bash
# Recount your actual components
ls plugins/compound-engineering/agents/*/*.md | wc -l
ls -d plugins/compound-engineering/skills/*/ | wc -l
ls plugins/compound-engineering/commands/*.md | wc -l

# Update these files with correct counts:
# - plugins/compound-engineering/.claude-plugin/plugin.json
# - .claude-plugin/marketplace.json
# - plugins/compound-engineering/README.md
```

### Deleted Component References

If upstream references components you've deleted (Rails/Python reviewers), remove those references during merge:

```bash
# Search for deleted component references
grep -r "kieran-rails-reviewer\|dhh-rails-reviewer\|kieran-python-reviewer" plugins/compound-engineering/

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
# 1. Validate JSON files
cat .claude-plugin/marketplace.json | jq .
cat plugins/compound-engineering/.claude-plugin/plugin.json | jq .

# 2. Verify counts are accurate
ls plugins/compound-engineering/agents/*/*.md | wc -l
ls -d plugins/compound-engineering/skills/*/ | wc -l

# 3. Check for broken references
grep -r "kieran-rails-reviewer\|dhh-rails-reviewer\|kieran-python-reviewer\|dhh-rails-style\|andrew-kane-gem-writer\|dspy-ruby" plugins/compound-engineering/

# 4. Test plugin installation
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
