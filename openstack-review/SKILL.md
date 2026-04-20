---
name: openstack-review
description: Review OpenStack patches from Gerrit with standardized analysis and inline comments. Use this skill whenever the user mentions OpenStack-specific review requests like "review this openstack ticket <number>", "review openstack patch <number>", "check openstack change <number>", or working with OpenStack projects (oslo.*, nova, neutron, etc.) and Gerrit. Also trigger for phrases like "review this gerrit change", "analyze this openstack patch", "check change <number> on gerrit", or any OpenStack code review workflow. Make sure to use this skill for OpenStack Gerrit reviews, not generic GitHub pull requests.
---

# OpenStack Gerrit Patch Review

This skill helps review OpenStack patches from Gerrit with comprehensive analysis and properly formatted output for posting comments back to Gerrit.

## When to use this skill

Trigger when the user:
- Mentions a Gerrit change ID (e.g., "review 981750")
- Asks to review a patch, change, or Gerrit review
- Wants to analyze an OpenStack code change

## Workflow

### 1. Extract the change ID

Parse the user's message to find the Gerrit change ID. It's typically a number (e.g., 981750).

### 2. Detect the OpenStack project

Determine which OpenStack project you're working with by checking:
- The current directory name
- Git remote URL
- Project configuration files (setup.cfg, .gitreview, etc.)

This context helps tailor the review to project-specific conventions.

### 3. Fetch the patch

Use git review to download the patch:

```bash
git review -d <change-id>
```

This will check out the patch as a local branch.

### 4. Analyze the changes

Perform a comprehensive review across multiple dimensions:

#### Code Quality
- Logic errors and potential bugs
- Edge cases not handled
- Error handling gaps
- Security vulnerabilities (SQL injection, XSS, command injection, etc.)
- Race conditions or concurrency issues
- Resource leaks (files, connections, memory)
- Code clarity and maintainability

#### OpenStack Conventions
- Adherence to OpenStack hacking rules
- Proper use of Oslo libraries and patterns
- API compatibility and versioning
- Configuration option handling
- Logging best practices (use of LOG, appropriate levels)
- Exception handling patterns

#### Tests
- Are there tests for the changes?
- Do tests cover edge cases?
- Are tests meaningful (not just for coverage)?
- Mock usage appropriateness
- Test naming and organization

#### Documentation
- Docstrings for new/modified functions
- Release notes (if needed)
- Inline comments for complex logic
- API documentation updates

#### Commit Message
- Proper format (summary line, blank line, detailed description)
- Clear explanation of the problem and solution
- References to bugs or blueprints
- Change-Id present

### 5. Generate the review report

Create a plain text report with this structure:

```
## Review Summary

[Brief overview of the patch - what it does, overall assessment]

### Statistics
- Files changed: X
- Lines added: Y
- Lines removed: Z

### Overall Assessment
[High-level feedback - is this ready to merge, needs work, etc.]

---

## Issues by Severity

### BLOCKING
[Critical issues that must be fixed before merge]

### SUGGESTIONS
[Improvements that should be considered]

### NITS
[Minor style/formatting issues]

---

## Detailed Inline Comments

<filename>:<line-number>: [SEVERITY] <issue description>

Example:
oslo_utils/timeutils.py:42: [BLOCKING] This function doesn't handle timezone-naive datetimes, which could cause a TypeError when calling .astimezone()

oslo_utils/tests/test_timeutils.py:15: [SUGGESTION] Consider adding a test case for negative time deltas

setup.cfg:8: [NIT] Alphabetize dependencies for consistency
```

**Important formatting rules:**
- Use plain text only (no markdown formatting)
- One issue per line in the inline comments section
- Use the format: `<filename>:<line>: [SEVERITY] <description>`
- Severity levels: BLOCKING, SUGGESTION, NIT
- Group issues by severity in the categorized section
- Keep inline comments concise and actionable
- The report should be ready to copy-paste into Gerrit

### 6. Provide context for the user

After generating the report:
- Explain how to post the review to Gerrit (e.g., using `git review` or the web UI)
- Mention any commands needed to return to the previous branch
- Offer to refine the review if needed

## Tips for effective reviews

- Read the commit message first to understand the intent
- Look at the diff holistically before diving into details
- Consider the context of the broader codebase
- Be constructive - suggest improvements, not just problems
- Distinguish between "must fix" and "nice to have"
- If you're unsure about a pattern, acknowledge that in your comment
- Check that the solution actually solves the stated problem

## Handling edge cases

- If `git review -d` fails, check if git-review is installed and the repository is configured
- If the change ID doesn't exist, provide a helpful error message
- If you're not in an OpenStack repository, warn the user
- If the patch is too large to review in one pass, break it down by file or concern area
