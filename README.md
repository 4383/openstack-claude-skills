# OpenStack Claude Skills

Custom Claude Code skills for OpenStack development workflows.

## About

This repository contains custom skills for [Claude Code](https://claude.ai/code) that enhance the development experience when working with OpenStack projects.

## Skills

### openstack-review

Review OpenStack patches from Gerrit with standardized analysis and inline comments.

**Usage:**
```
review this openstack patch <change-id>
```

**Features:**
- Fetches patches directly from OpenStack's Gerrit (review.opendev.org)
- Performs comprehensive code review across multiple dimensions:
  - Code quality and logic errors
  - OpenStack-specific conventions (hacking rules, Oslo patterns)
  - Test coverage and quality
  - Documentation completeness
  - Commit message format
- Generates structured review reports with severity levels (BLOCKING, SUGGESTION, NIT)
- Provides actionable inline comments ready to post back to Gerrit

**Examples:**
- `review this openstack patch 978095`
- `review openstack change 123456`
- `check this gerrit patch 999999`

## Installation

### Prerequisites

- [Claude Code](https://claude.ai/code) installed
- Git

### Setup

1. Clone this repository:
   ```bash
   git clone <your-repo-url> ~/dev/openstack-claude-skills
   ```

2. Create symlinks to Claude's skills directory:
   ```bash
   ln -s ~/dev/openstack-claude-skills/openstack-review ~/.claude/skills/openstack-review
   ```

3. Restart Claude Code or reload skills (skills are loaded automatically)

### Verify Installation

After installation, the skills should appear in Claude Code's available skills list. You can verify by asking Claude to review an OpenStack patch.

## Development

### Adding New Skills

1. Create a new directory in this repository:
   ```bash
   cd ~/dev/openstack-claude-skills
   mkdir my-new-skill
   ```

2. Create the skill file following Claude Code's skill format

3. Symlink to Claude's directory:
   ```bash
   ln -s ~/dev/openstack-claude-skills/my-new-skill ~/.claude/skills/my-new-skill
   ```

4. Commit and push:
   ```bash
   git add my-new-skill/
   git commit -m "Add my-new-skill"
   git push
   ```

### Skill Format

Skills should follow the Claude Code skill specification. See the [openstack-review](./openstack-review/) skill as an example.

## Contributing

Contributions are welcome! Please:

1. Fork this repository
2. Create a feature branch
3. Test your changes thoroughly
4. Submit a pull request

## License

[Add your license here]

## Author

Hervé Beraud

## Acknowledgments

- OpenStack community for the development workflows this automates
- Anthropic for Claude Code and the skills framework
