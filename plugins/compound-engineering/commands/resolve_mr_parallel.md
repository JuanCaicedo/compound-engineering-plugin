---
name: resolve_mr_parallel
description: Resolve all MR comments using parallel processing
argument-hint: "[optional: MR number or current MR]"
---

Resolve all MR comments using parallel processing.

Claude Code automatically detects and understands your git context:

- Current branch detection
- Associated MR context
- All MR comments and review threads
- Can work with any MR by specifying the MR number, or ask it.

## Workflow

### 1. Analyze

Get all unresolved comments for MR

```bash
glab mr view
bin/get-mr-comments MR_NUMBER
```

### 2. Plan

Create a TodoWrite list of all unresolved items grouped by type.

### 3. Implement (PARALLEL)

Spawn a mr-comment-resolver agent for each unresolved item in parallel.

So if there are 3 comments, it will spawn 3 mr-comment-resolver agents in parallel. like this

1. Task mr-comment-resolver(comment1)
2. Task mr-comment-resolver(comment2)
3. Task mr-comment-resolver(comment3)

Always run all in parallel subagents/Tasks for each Todo item.

### 4. Commit & Resolve

- Commit changes
- Run bin/resolve-mr-thread THREAD_ID_1
- Push to remote

Last, check bin/get-mr-comments MR_NUMBER again to see if all comments are resolved. They should be, if not, repeat the process from 1.
