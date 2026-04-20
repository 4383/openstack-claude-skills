# OpenStack Claude Skills

This repository contains custom Claude Code skills for OpenStack development.

## Repository Structure

```
openstack-claude-skills/
├── openstack-review/          # Skill for reviewing OpenStack Gerrit patches
│   └── skill file(s)
├── README.md                  # User documentation
└── CLAUDE.md                  # This file - development guide for Claude
```

## Purpose

This repository provides reusable automation for common OpenStack development tasks through Claude Code skills. Skills are designed to be:

1. **OpenStack-aware**: Understanding of OpenStack conventions, tools (Gerrit, Zuul, etc.), and project structure
2. **Workflow-optimized**: Streamlining repetitive tasks like code reviews, patch analysis
3. **Standardized**: Following OpenStack community standards and best practices

## Skills Overview

### openstack-review

**Purpose:** Automate comprehensive code review of OpenStack Gerrit patches

**Key Capabilities:**
- Fetches patches from review.opendev.org via Gerrit API
- Analyzes code quality, OpenStack conventions, tests, and documentation
- Generates structured review reports with severity-based issue categorization
- Provides inline comments with file:line references

**Usage Pattern:**
```
User: review this openstack patch 978095
Claude: [Invokes skill] → Fetches patch → Analyzes → Returns formatted review
```

**Implementation Notes:**
- Uses WebFetch tool to access Gerrit REST API
- Handles Gerrit's JSON security prefix (`)]}'\n`)
- Parses diffs and commit metadata
- Applies OpenStack-specific review criteria (hacking rules, Oslo patterns, etc.)

## Development Guidelines

### When Creating New Skills

1. **Scope**: Focus on OpenStack-specific workflows that benefit from automation
2. **Naming**: Use descriptive, action-oriented names (e.g., `openstack-review`, not `review-tool`)
3. **Documentation**: Include clear trigger patterns and usage examples
4. **Testing**: Test with real OpenStack patches before committing

### Skill Trigger Patterns

Skills should trigger on clear, unambiguous patterns:
- ✅ "review this openstack patch 123456"
- ✅ "check openstack change 789101"
- ❌ "review this" (too generic)

### OpenStack-Specific Context

Skills should understand:
- **Gerrit workflow**: change IDs, revisions, review labels (+2/-2, Workflow+1)
- **Project structure**: oslo.* libraries, service projects, governance
- **Tools**: git-review, tox, hacking, pbr
- **Community standards**: commit message format, testing requirements, documentation

### Error Handling

- Gracefully handle network failures when fetching from Gerrit
- Provide helpful messages when patches aren't found
- Fall back to alternative approaches when primary methods fail

### API Usage

When accessing OpenStack infrastructure:
- Use public APIs (Gerrit REST, Zuul API)
- Don't require authentication for read-only operations
- Respect rate limits
- Cache responses when appropriate

## Common Patterns

### Fetching from Gerrit

```
Gerrit JSON responses start with:
)]}'
{"actual": "json", "data": "here"}

Strip the first line before parsing.
```

### Gerrit REST API Endpoints

- Change details: `https://review.opendev.org/changes/{change-id}`
- Files in revision: `https://review.opendev.org/changes/{change-id}/revisions/current/files`
- Diff for file: `https://review.opendev.org/changes/{change-id}/revisions/current/files/{encoded-path}/diff`
- Commit message: `https://review.opendev.org/changes/{change-id}/revisions/current/commit`

### Review Output Format

Reviews should follow this structure:
```
## Review Summary
[Brief overview]

### Statistics
- Files changed: X
- Lines added: Y
- Lines removed: Z

### Overall Assessment
[High-level feedback]

---

## Issues by Severity

### BLOCKING
[Critical issues]

### SUGGESTIONS
[Improvements to consider]

### NITS
[Minor issues]

---

## Detailed Inline Comments

file.py:42: [BLOCKING] Description of issue
file.py:100: [SUGGESTION] Suggested improvement
```

## Maintenance

### Updating Skills

1. Test changes locally first
2. Update skill documentation if behavior changes
3. Consider backward compatibility
4. Commit with clear, descriptive messages

### Version Control

- Commit working skills, not work-in-progress
- Use meaningful commit messages following OpenStack conventions
- Tag releases if skills are shared widely

### Symlink Management

Skills are symlinked from `~/.claude/skills/` to this repository:
```bash
~/.claude/skills/openstack-review -> ~/dev/openstack-claude-skills/openstack-review
```

To update, just edit files in this repository - changes are immediately available to Claude Code.

## Future Skill Ideas

Potential skills to implement:
- `openstack-test-runner`: Run specific test suites with proper tox environment
- `openstack-release-notes`: Generate release note templates
- `openstack-depends-on`: Analyze cross-repo dependencies
- `openstack-ci-debug`: Parse Zuul logs for failures

## Resources

- [OpenStack Developer Documentation](https://docs.openstack.org/contributors/)
- [Gerrit Code Review](https://review.opendev.org/)
- [OpenStack Hacking Guidelines](https://docs.openstack.org/hacking/latest/)
- [Claude Code Skills Documentation](https://github.com/anthropics/claude-code)
