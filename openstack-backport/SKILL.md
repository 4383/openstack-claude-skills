---
name: openstack-backport
description: Analyze OpenStack commits to identify backport candidates for stable branches based on OpenStack backport policy. Use this skill whenever the user mentions backporting, stable branches, checking what needs backporting, analyzing commits for backport eligibility, finding backport candidates, or working with OpenStack release branches. Also trigger when users ask about "what commits need to go to stable/X", "check backports since release Y", "find bugs that should be backported", or any variation of identifying which changes should be cherry-picked to maintained stable branches.
---

# OpenStack Backport Checker

This skill analyzes commits on the master branch of an OpenStack repository to identify which commits should be backported to maintained stable branches according to OpenStack backport policy.

## OpenStack Backport Policy

### Backport Candidates
- **Critical and High priority bug fixes** - Must be reported in Launchpad
- The bug must have priority "Critical" or "High" in Launchpad

### Backport Exclusions
- Medium and Low priority bug fixes
- New features
- Commits without bug references

## Workflow

### 1. Determine Maintained Branches

Fetch the series status from OpenStack releases repository:
```
https://opendev.org/openstack/releases/raw/branch/master/data/series_status.yaml
```

Parse this YAML to identify branches with status `maintained` or `extended maintenance`. These are the target branches for backports.

Example status values:
- `development` - Current development branch (master)
- `maintained` - Actively maintained stable branch
- `extended maintenance` - Extended support
- `unmaintained` - No longer supported
- `end of life` - EOL

Only consider branches with `maintained` or `extended maintenance` status as backport targets.

### 2. Identify Commit Range

Support flexible commit range specification:

**By count**: "check last 10 commits"
```bash
git log -n 10 --oneline
```

**Since release/tag**: "check commits since 2024.1"
```bash
git log <tag>..HEAD --oneline
```

**Since date**: "check commits since 2024-01-01"
```bash
git log --since="2024-01-01" --oneline
```

**Between tags**: "check commits between 2024.1 and 2024.2"
```bash
git log <tag1>..<tag2> --oneline
```

**Unreleased commits**: "check unreleased commits"
```bash
# Get latest tag
latest_tag=$(git describe --tags --abbrev=0)
git log ${latest_tag}..HEAD --oneline
```

If the user doesn't specify a range, default to unreleased commits since the last tag.

### 3. Parse Commits for Bug References

For each commit, extract bug references from the commit message. OpenStack uses these formats:

- `Closes-Bug: #1234567` - This commit completely fixes the bug
- `Partial-Bug: #1234567` - This commit partially addresses the bug
- `Related-Bug: #1234567` - This commit is related to the bug

Also check for:
- `Closes-Bug: https://bugs.launchpad.net/project/+bug/1234567`
- Multiple bug references in one commit

Use regex patterns to extract bug IDs:
```
Closes-Bug:\s*#?(\d+)
Partial-Bug:\s*#?(\d+)
Related-Bug:\s*#?(\d+)
Closes-Bug:.*bugs\.launchpad\.net/.*/\+bug/(\d+)
```

### 4. Query Launchpad API

For each bug ID found, query the Launchpad API to get bug priority:

```
https://api.launchpad.net/1.0/bugs/{bug_id}
```

The response is JSON. Extract the `importance` field:
```json
{
  "importance": "High",
  "title": "Bug title",
  "status": "Fix Committed",
  ...
}
```

Possible importance values:
- `Critical`
- `High`
- `Medium`
- `Low`
- `Wishlist`
- `Undecided`

**Backport qualification**: Only `Critical` and `High` bugs qualify for backport.

Handle errors gracefully:
- If bug is private (403), note it but skip
- If bug doesn't exist (404), note it but skip
- If network fails, report the error clearly

### 5. Check if Commit Already Backported

For each maintained branch, check if the commit has already been backported:

```bash
git branch -r | grep "origin/stable/"
```

For each stable branch:
```bash
git log origin/stable/<branch> --grep="<commit-sha>" --all-match
```

Or check if the commit exists in the branch:
```bash
git branch -r --contains <commit-sha>
```

Also look for cherry-pick references in commit messages on stable branches that reference the original commit SHA.

### 6. Generate Report

Structure the output in two views:

#### View 1: By Commit

List each commit that qualifies for backport, showing:
- Commit SHA (short form)
- Commit date
- Author
- Commit message (first line)
- Bug ID(s) and priority
- Which stable branches need this commit
- Cherry-pick command suggestion

#### View 2: By Branch

Group commits by target stable branch, showing:
- Branch name
- Number of commits needing backport
- List of commit SHAs
- Bulk cherry-pick commands

Include summary statistics:
- Total commits analyzed
- Total backport candidates
- Commits per branch
- Most common bug priorities

## Output Format

Use this template:

```markdown
# OpenStack Backport Analysis

**Repository**: <repo-name>
**Analysis Date**: <date>
**Commit Range**: <range-description>

## Summary

- Total commits analyzed: X
- Backport candidates: Y
- Maintained stable branches: Z
- Critical bugs: A
- High bugs: B

---

## Maintained Branches

| Branch | Status | Commits Needed |
|--------|--------|----------------|
| stable/2024.1 | maintained | 5 |
| stable/2023.2 | extended maintenance | 3 |

---

## Backport Candidates (By Commit)

### Commit: abc1234 - Fix critical memory leak in scheduler
- **Date**: 2024-04-15
- **Author**: Jane Developer
- **Bug**: [LP#1234567](https://bugs.launchpad.net/project/+bug/1234567) - **Priority: Critical**
- **Target Branches**: stable/2024.1, stable/2023.2
- **Already Backported**: No
- **Cherry-pick**:
  ```bash
  git cherry-pick abc1234
  ```

### Commit: def5678 - Fix high priority race condition
- **Date**: 2024-04-10
- **Author**: John Maintainer
- **Bug**: [LP#2345678](https://bugs.launchpad.net/project/+bug/2345678) - **Priority: High**
- **Target Branches**: stable/2024.1
- **Already Backported**: stable/2023.2
- **Cherry-pick**:
  ```bash
  git cherry-pick def5678
  ```

---

## Backport Plan (By Branch)

### stable/2024.1 (5 commits needed)

```bash
# Cherry-pick all commits for stable/2024.1
git checkout stable/2024.1
git cherry-pick abc1234
git cherry-pick def5678
git cherry-pick ghi9012
git cherry-pick jkl3456
git cherry-pick mno7890
```

**Commits**:
1. abc1234 - Fix critical memory leak (Critical)
2. def5678 - Fix race condition (High)
3. ghi9012 - Fix database deadlock (Critical)
4. jkl3456 - Fix API timeout (High)
5. mno7890 - Fix config parsing (High)

### stable/2023.2 (3 commits needed)

```bash
# Cherry-pick all commits for stable/2023.2
git checkout stable/2023.2
git cherry-pick abc1234
git cherry-pick ghi9012
git cherry-pick mno7890
```

**Commits**:
1. abc1234 - Fix critical memory leak (Critical)
2. ghi9012 - Fix database deadlock (Critical)
3. mno7890 - Fix config parsing (High)

---

## Excluded Commits

### Medium/Low Priority (X commits)
- pqr1234 - LP#3456789 (Medium): Fix minor logging issue
- stu5678 - LP#4567890 (Low): Improve error message

### No Bug Reference (Y commits)
- vwx9012 - Refactor utility function (no bug reference)

### Features (Z commits)
- yza3456 - Add new API endpoint (feature, not eligible)

---

## Notes

- All backport candidates have been verified against Launchpad
- Commits already backported to some branches are noted
- Use `git log <branch>` to verify before cherry-picking
- Test each cherry-pick in a local environment before proposing
```

## Error Handling

Handle common errors gracefully:

1. **Not in a git repository**: Clearly state "This directory is not a git repository"
2. **No commits in range**: "No commits found in specified range"
3. **Launchpad API failures**: Note which bugs couldn't be checked and continue
4. **Series status fetch fails**: Explain that branch status couldn't be determined
5. **No bug references found**: List commits without bug references in "Excluded" section

## Important Notes

- The Launchpad API is public and doesn't require authentication for reading bug data
- Be patient with Launchpad queries - rate limit if checking many bugs
- Cache bug priority data during the analysis to avoid duplicate API calls
- Git commands should be run from the repository root
- When showing cherry-pick commands, use the short SHA (7 characters) for readability
- Include the full commit message first line to help reviewers
- Always check if commits are already backported to avoid duplicate work
- Consider commit dependencies - if commit B depends on commit A, note this in the report

## Example Usage

**Example 1**: Check last 20 commits
```
User: check the last 20 commits for backport candidates
```

**Example 2**: Check since a specific release
```
User: what needs backporting to stable branches since 2024.1?
```

**Example 3**: Check specific date range
```
User: find backport candidates from the last month
```

**Example 4**: Target specific branch
```
User: show me what commits should go to stable/2024.1
```

In all cases:
1. Fetch maintained branch list from series_status.yaml
2. Get commits in the specified range
3. Parse bug references from each commit
4. Query Launchpad for bug priorities
5. Check if commits are already backported
6. Generate the structured report

The report should be comprehensive but scannable - use tables, clear headings, and actionable commands so maintainers can quickly act on the findings.
