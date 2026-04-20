# OpenStack Claude Skills

Custom Claude Code skills for OpenStack development workflows.

## Table of Contents

- [About](#about)
- [Skills](#skills)
  - [openstack-review](#openstack-review)
  - [openstack-backport](#openstack-backport)
- [Installation](#installation)
  - [Method 1: Plugin Marketplace (Recommended)](#method-1-plugin-marketplace-recommended)
  - [Method 2: Manual Installation (Development)](#method-2-manual-installation-development)
  - [Method 3: Local Marketplace](#method-3-local-marketplace)
  - [Verify Installation](#verify-installation)
- [Development](#development)
  - [Adding New Skills](#adding-new-skills)
  - [Skill Format](#skill-format)
- [Contributing](#contributing)
- [License](#license)
- [Author](#author)
- [Acknowledgments](#acknowledgments)

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

### openstack-backport

Analyze commits to identify backport candidates for OpenStack stable branches based on the official backport policy.

**Usage:**
```
check backports for the last 15 commits
what needs backporting to stable branches since 2024.1?
find backport candidates from the last month
```

**Features:**
- Fetches maintained branch status from OpenStack releases repository
- Supports flexible commit range specification (by count, tag, date, or range)
- Parses bug references from OpenStack commit messages (Closes-Bug, Partial-Bug, Related-Bug)
- Queries Launchpad API to verify bug priorities
- Enforces OpenStack backport policy (only Critical/High priority bugs qualify)
- Checks if commits are already backported to avoid duplicate work
- Generates comprehensive reports with:
  - Summary statistics
  - Backport candidates grouped by commit and by branch
  - Excluded commits with reasons (Medium/Low priority, no bug reference, features)
  - Executable cherry-pick commands for each stable branch

**Examples:**
- `check the last 20 commits for backport candidates`
- `what needs backporting to stable branches since 2024.1?`
- `find backport candidates between 2023.2 and HEAD`
- `show me what commits should go to stable/2024.1`

## Installation

### Method 1: Plugin Marketplace (Recommended)

If this repository is hosted on GitHub, you can install skills via the plugin marketplace:

1. Add the marketplace to Claude Code:
   ```
   /plugin marketplace add <github-username>/openstack-claude-skills
   ```

2. Install skills:
   ```
   /plugin install openstack-review@openstack-skills
   /plugin install openstack-backport@openstack-skills
   ```

### Method 2: Manual Installation (Development)

For local development or if you want to modify the skills:

1. Clone this repository:
   ```bash
   git clone <your-repo-url> ~/dev/openstack-claude-skills
   ```

2. Create symlinks to Claude's skills directory:
   ```bash
   ln -s ~/dev/openstack-claude-skills/openstack-review ~/.claude/skills/openstack-review
   ln -s ~/dev/openstack-claude-skills/openstack-backport ~/.claude/skills/openstack-backport
   ```

3. Restart Claude Code or reload skills (skills are loaded automatically)

### Method 3: Local Marketplace

You can also add your local repository as a marketplace:

1. Edit your Claude Code settings (`~/.claude/settings.json`):
   ```json
   {
     "extraKnownMarketplaces": {
       "openstack-skills": {
         "source": {
           "source": "local",
           "path": "/home/your-username/dev/openstack-claude-skills"
         }
       }
     }
   }
   ```

2. Install skills:
   ```
   /plugin install openstack-review@openstack-skills
   /plugin install openstack-backport@openstack-skills
   ```

### Verify Installation

After installation, the skills should appear in Claude Code's available skills list. You can verify by:
- Asking Claude to review an OpenStack patch
- Asking Claude to check backport candidates in an OpenStack repository

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

This project is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.

## Author

Hervé Beraud

## Acknowledgments

- OpenStack community for the development workflows this automates
- Anthropic for Claude Code and the skills framework
