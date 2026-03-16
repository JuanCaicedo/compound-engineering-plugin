# Compounding Engineering Plugin

> **Fork Information:** This is Juan Caicedo's fork with Rails, Ruby, and Python components removed. Original by Kieran Klaassen at [EveryInc/compound-engineering-plugin](https://github.com/EveryInc/compound-engineering-plugin).

AI-powered development tools that get smarter with every use. Make each unit of engineering work easier than the last.

## Components

| Component | Count |
|-----------|-------|
| Agents | 27 |
| Commands | 2 |
| Skills | 45 |
| MCP Servers | 1 |

## Agents

Agents are organized into categories for easier discovery.

### Review (14)

| Agent | Description |
|-------|-------------|
| `agent-native-reviewer` | Verify features are agent-native (action + context parity) |
| `architecture-strategist` | Analyze architectural decisions and compliance |
| `brandon-aldrich-reviewer` | Architecture, performance, and component design review |
| `code-simplicity-reviewer` | Final pass for simplicity and minimalism |
| `data-integrity-guardian` | Database migrations and data integrity |
| `data-migration-expert` | Validate ID mappings match production, check for swapped values |
| `deployment-verification-agent` | Create Go/No-Go deployment checklists for risky data changes |
| `jeremy-gillick-reviewer` | Readability, code organization, and React performance review |
| `julik-frontend-races-reviewer` | Review JavaScript/Stimulus code for race conditions |
| `kieran-typescript-reviewer` | TypeScript code review with strict conventions |
| `pattern-recognition-specialist` | Analyze code for patterns and anti-patterns |
| `performance-oracle` | Performance analysis and optimization |
| `schema-drift-detector` | Detect unrelated schema.rb changes in PRs |
| `security-sentinel` | Security audits and vulnerability assessments |

### Research (5)

| Agent | Description |
|-------|-------------|
| `best-practices-researcher` | Gather external best practices and examples |
| `framework-docs-researcher` | Research framework documentation and best practices |
| `git-history-analyzer` | Analyze git history and code evolution |
| `learnings-researcher` | Search institutional learnings for relevant past solutions |
| `repo-research-analyst` | Research repository structure and conventions |

### Design (3)

| Agent | Description |
|-------|-------------|
| `design-implementation-reviewer` | Verify UI implementations match Figma designs |
| `design-iterator` | Iteratively refine UI through systematic design iterations |
| `figma-design-sync` | Synchronize web implementations with Figma designs |

### Workflow (4)

| Agent | Description |
|-------|-------------|
| `bug-reproduction-validator` | Systematically reproduce and validate bug reports |
| `lint` | Run linting and code quality checks on Ruby and ERB files |
| `mr-comment-resolver` | Address MR comments and implement fixes |
| `spec-flow-analyzer` | Analyze user flows and identify gaps in specifications |

### Docs (1)

| Agent | Description |
|-------|-------------|
| `ankane-readme-writer` | Create READMEs following Ankane-style template for Ruby gems |

## Commands

| Command | Description |
|---------|-------------|
| `/agent_review` | Invoke a specific reviewer agent for focused code review |
| `/resolve_mr_parallel` | Resolve MR comments using parallel processing |

## Skills

> **Note:** Most former commands were migrated to skills in v2.39.0. They work identically as `/skill-name`.

### Core Workflows

Core workflow skills use `ce-` prefix (invoked as `/ce:brainstorm`, `/ce:plan`, etc.):

| Skill | Description |
|-------|-------------|
| `ce-brainstorm` | Explore requirements and approaches before planning |
| `ce-plan` | Create implementation plans |
| `ce-review` | Run comprehensive code reviews |
| `ce-work` | Execute work items systematically |
| `ce-compound` | Document solved problems to compound team knowledge |

> **Deprecated aliases:** `workflows-*` skills still work as aliases that forward to `ce-*` equivalents.

### Automation & Orchestration

| Skill | Description |
|-------|-------------|
| `lfg` | Full autonomous engineering workflow |
| `slfg` | Full autonomous workflow with swarm mode |
| `deepen-plan` | Enhance plans with parallel research agents |
| `orchestrating-swarms` | Guide to multi-agent swarm orchestration |
| `resolve_parallel` | Resolve TODO comments in parallel |
| `resolve_todo_parallel` | Resolve todos in parallel |
| `resolve-pr-parallel` | Resolve PR review comments in parallel |

### Development Tools

| Skill | Description |
|-------|-------------|
| `changelog` | Create engaging changelogs for recent merges |
| `create-agent-skill` | Create or edit Claude Code skills |
| `create-agent-skills` | Expert guidance for creating Claude Code skills |
| `generate_command` | Generate new slash commands |
| `heal-skill` | Fix skill documentation issues |
| `compound-docs` | Capture solved problems as categorized documentation |
| `frontend-design` | Create production-grade frontend interfaces |

### Testing & Browser

| Skill | Description |
|-------|-------------|
| `test-browser` | Run browser tests on MR-affected pages |
| `test-xcode` | Build and test iOS apps on simulator |
| `agent-browser` | CLI-based browser automation using Vercel's agent-browser |
| `reproduce-bug` | Reproduce bugs using logs and console |

### Architecture & Design

| Skill | Description |
|-------|-------------|
| `agent-native-architecture` | Build AI agents using prompt-native architecture |
| `agent-native-audit` | Comprehensive agent-native architecture review |

### Planning & Review

| Skill | Description |
|-------|-------------|
| `brainstorming` | Explore requirements and approaches through collaborative dialogue |
| `generate-review-guide` | Generate comprehensive reviewer's guides from GitLab MR URLs |
| `document-review` | Improve documents through structured self-review |
| `setup` | Configure which review agents run for your project |

### Content & Workflow

| Skill | Description |
|-------|-------------|
| `every-style-editor` | Review copy for Every's style guide compliance |
| `file-todos` | File-based todo tracking system |
| `git-worktree` | Manage Git worktrees for parallel development |
| `keyboard-troubleshooting` | Diagnose and fix keyboard input issues |
| `proof` | Create, edit, and share documents via Proof collaborative editor |
| `triage` | Triage and prioritize issues |
| `report-bug` | Report a bug in the plugin |
| `feature-video` | Record video walkthroughs and add to MR description |
| `deploy-docs` | Deploy documentation site |

### File Transfer & Image

| Skill | Description |
|-------|-------------|
| `rclone` | Upload files to S3, Cloudflare R2, Backblaze B2, and cloud storage |
| `gemini-imagegen` | Generate and edit images using Google's Gemini API |

**gemini-imagegen features:**
- Text-to-image generation
- Image editing and manipulation
- Multi-turn refinement
- Multiple reference image composition (up to 14 images)

**Requirements:**
- `GEMINI_API_KEY` environment variable
- Python packages: `google-genai`, `pillow`

## MCP Servers

| Server | Description |
|--------|-------------|
| `context7` | Framework documentation lookup via Context7 |

### Context7

**Tools provided:**
- `resolve-library-id` - Find library ID for a framework/package
- `get-library-docs` - Get documentation for a specific library

Supports 100+ frameworks including Rails, React, Next.js, Vue, Django, Laravel, and more.

MCP servers start automatically when the plugin is enabled.

**Authentication:** To avoid anonymous rate limits, set the `CONTEXT7_API_KEY` environment variable with your Context7 API key. The plugin passes this automatically via the `x-api-key` header. Without it, requests go unauthenticated and will quickly hit the anonymous quota limit.

## Browser Automation

This plugin uses **agent-browser CLI** for browser automation tasks. Install it globally:

```bash
npm install -g agent-browser
agent-browser install  # Downloads Chromium
```

The `agent-browser` skill provides comprehensive documentation on usage.

## Installation

```bash
claude /plugin install compound-engineering
```

## Known Issues

### MCP Servers Not Auto-Loading

**Issue:** The bundled Context7 MCP server may not load automatically when the plugin is installed.

**Workaround:** Manually add it to your project's `.claude/settings.json`:

```json
{
  "mcpServers": {
    "context7": {
      "type": "http",
      "url": "https://mcp.context7.com/mcp",
      "headers": {
        "x-api-key": "${CONTEXT7_API_KEY:-}"
      }
    }
  }
}
```

Set `CONTEXT7_API_KEY` in your environment to authenticate. Or add it globally in `~/.claude/settings.json` for all projects.

## Version History

See [CHANGELOG.md](CHANGELOG.md) for detailed version history.

## License

MIT
