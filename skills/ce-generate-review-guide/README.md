# Generating MR Reviews Skill

Generates comprehensive reviewer's guides from GitLab merge request URLs.

## Usage

Trigger this skill with any of these phrases:
- "Generate a reviewer's guide for {gitlab-url}"
- "Create review guide for MR {number}"
- "Review this MR: {url}"
- "Analyze this merge request"

## What It Does

1. **Fetches MR data** using `glab` CLI
2. **Analyzes changes** to understand scope and impact
3. **Reads key files** to understand architecture
4. **Generates structured guide** with:
   - Overview and context
   - Architecture breakdown
   - Testing checklists
   - Prioritized file list
   - Questions for author
   - Deployment considerations
5. **Saves to** `~/code/vanna/docs/reviews/MR-{number}-{slug}.md`

## Output Structure

The generated review guide includes:

- **Overview**: Links, description, demo video
- **Key Changes**: Summary of additions/modifications
- **Architecture**: Visual breakdown of structure
- **Review Checklist**: Critical areas, testing scenarios, code quality
- **File Priorities**: Must review, should review, can skim
- **Questions**: Clarifying questions for the author
- **Testing**: Functional, edge case, performance tests
- **Deployment**: Migration, rollback, monitoring considerations

## Examples

### Basic Usage
```
User: "Generate review guide for https://gitlab.com/vanna-health/vanna-core/vanna-connect/-/merge_requests/1929"
```

Creates: `~/code/vanna/docs/reviews/MR-1929-referrals-home.md`

### Short Form
```
User: "Review MR 1845"
```

Creates: `~/code/vanna/docs/reviews/MR-1845-{title-slug}.md`

## Customization

The skill automatically:
- Adjusts depth based on MR size (<10 files vs >50 files)
- Includes relevant sections (DB migrations, performance, security)
- Customizes testing scenarios by MR type (feature/bugfix/refactor)
- Prioritizes files by criticality

## Requirements

- `glab` CLI tool installed and configured
- Access to the GitLab repository
- Git repository context

## File Structure

```
generate-review-guide/
├── SKILL.md         # Main skill definition
└── README.md        # This file
```

## Success Criteria

A good review guide:
- ✅ Links to MR and related issues
- ✅ Explains architecture clearly
- ✅ Provides actionable test cases
- ✅ Prioritizes files effectively
- ✅ Asks clarifying questions
- ✅ Considers deployment impact
