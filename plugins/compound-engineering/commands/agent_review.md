---
name: agent_review
description: Invoke a specific reviewer agent for focused code review
argument-hint: "<reviewer-name> [optional: file path or PR/MR number]"
---

# Agent Review

Launch a specific reviewer agent to analyze code with their particular focus and style.

## Usage

### Review with Specific Agent

```bash
/agent_review jeremy                    # Review with Jeremy Gillick's style
/agent_review brandon                   # Review with Brandon Aldrich's style
/agent_review kieran                    # Review with Kieran's TypeScript style
/agent_review security                  # Security-focused review
```

### Review Specific Files or PRs

```bash
/agent_review jeremy src/components/PatientList.tsx
/agent_review brandon 123              # Review MR #123
/agent_review kieran .                 # Review entire codebase
```

## Available Reviewers

### React/TypeScript Specialists

**jeremy** (jeremy-gillick-reviewer)
- Code organization and extraction
- React performance (useMemo/useCallback)
- Code duplication elimination
- Readability through clear variables
- File/component organization
- Test organization
- Style: Educational, uses praise:/nit:, asks guiding questions

**brandon** (brandon-aldrich-reviewer)
- Module boundaries and dependencies
- Performance optimization (batched queries)
- Component simplicity
- References to codebase patterns
- Visual regression awareness
- Style: Collaborative, uses Request/Question/Praise prefixes

**kieran** (kieran-typescript-reviewer)
- Strict TypeScript conventions
- Type safety and inference
- Code quality standards
- Best practices enforcement
- Style: High quality bar, direct feedback

**julik** (julik-frontend-races-reviewer)
- JavaScript/Stimulus race conditions
- DOM timing issues
- Asynchronous UI behavior
- Frontend data races
- Style: Eye for subtle timing bugs

### Architecture & Design

**architecture** (architecture-strategist)
- System design decisions
- Component boundaries
- Architectural compliance
- Design pattern evaluation

**simplicity** (code-simplicity-reviewer)
- Minimalism and YAGNI
- Removing unnecessary complexity
- Simplification opportunities
- Over-engineering detection

**patterns** (pattern-recognition-specialist)
- Design patterns and anti-patterns
- Naming conventions
- Code duplication
- Consistency analysis

### Security & Performance

**security** (security-sentinel)
- Security vulnerabilities
- OWASP compliance
- Input validation
- Secret exposure
- Authentication/authorization

**performance** (performance-oracle)
- Performance bottlenecks
- Algorithm optimization
- Memory usage
- Database query efficiency
- Caching strategies

### Data & Deployment

**data-integrity** (data-integrity-guardian)
- Database migrations safety
- Data constraints
- Transaction boundaries
- Referential integrity
- Privacy requirements

**data-migration** (data-migration-expert)
- ID mappings validation
- Production data verification
- Swapped values detection
- Rollback safety

**deployment** (deployment-verification-agent)
- Go/No-Go checklists
- Pre/post-deploy verification
- Rollback procedures
- Monitoring plans

### Quality & Standards

**agent-native** (agent-native-reviewer)
- Agent-native architecture
- Action parity
- Context accessibility
- Tool availability

## Implementation

When this command is invoked:

1. **Parse arguments**:
   - First argument: reviewer name (required)
   - Remaining arguments: file path, PR/MR number, or scope

2. **Map reviewer name to agent**:
   ```
   jeremy    → jeremy-gillick-reviewer
   brandon   → brandon-aldrich-reviewer
   kieran    → kieran-typescript-reviewer
   julik     → julik-frontend-races-reviewer
   security  → security-sentinel
   performance → performance-oracle
   architecture → architecture-strategist
   simplicity → code-simplicity-reviewer
   patterns  → pattern-recognition-specialist
   data-integrity → data-integrity-guardian
   data-migration → data-migration-expert
   deployment → deployment-verification-agent
   agent-native → agent-native-reviewer
   ```

3. **Determine review scope** from remaining arguments:
   - No additional args: Review uncommitted changes (git diff)
   - File path: Review specified file(s)
   - Number: Review MR/PR with that ID
   - ".": Review entire codebase (warn about scope)

4. **Gather context**:
   ```bash
   # For uncommitted changes
   git diff

   # For specific files
   cat [file-path]

   # For MR/PR
   gh pr view [number] --json files,diff
   # or
   glab mr view [number] --with-diff
   ```

5. **Invoke the reviewer agent**:
   ```
   Use the Task tool with:
   - subagent_type: [mapped-agent-name]
   - description: "Review code with [reviewer] focus"
   - prompt: "Review the following code changes:\n\n[paste diff/code here]\n\nProvide feedback focusing on [reviewer's expertise areas]."
   ```

6. **Present the review** to the user

## Error Handling

**Invalid reviewer name:**
```
Error: Unknown reviewer 'foo'

Available reviewers:
  React/TypeScript: jeremy, brandon, kieran, julik
  Architecture: architecture, simplicity, patterns
  Security: security, performance
  Data: data-integrity, data-migration, deployment
  Quality: agent-native

Usage: /agent_review <reviewer-name> [file-or-pr]
```

**Missing reviewer name:**
```
Error: Reviewer name required

Usage: /agent_review <reviewer-name> [file-or-pr]

Examples:
  /agent_review jeremy
  /agent_review brandon src/components/
  /agent_review security 123
```

## Examples

**Quick review of current changes:**
```bash
/agent_review jeremy
```

**Review specific component with Brandon's focus:**
```bash
/agent_review brandon src/modules/outreach/components/OutreachCard.tsx
```

**Security audit of PR:**
```bash
/agent_review security 456
```

**Full architecture review:**
```bash
/agent_review architecture .
```

## Tips for Best Results

- **Match reviewer to concern**: Use jeremy for React performance, security for vulnerabilities, etc.
- **Provide context**: Mention specific areas of concern in follow-up
- **Iterate**: Use feedback to improve, then review again with same or different reviewer
- **Combine reviewers**: Run multiple reviewers for comprehensive feedback
- **Ask questions**: If suggestions aren't clear, ask for clarification

## Related Commands

- `/workflows:review` - Multi-agent comprehensive review (all reviewers in parallel)
- `/plan_review` - Review implementation plans with multiple agents
- `/agent-native-audit` - Comprehensive agent-native architecture audit

## Success Criteria

- Correct reviewer agent invoked based on name
- Review scope determined correctly from arguments
- Feedback matches reviewer's style and focus areas
- Suggestions are specific and actionable
- Communication uses reviewer's authentic style
