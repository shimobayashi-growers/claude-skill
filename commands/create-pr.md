---
allowed-tools: Bash(gh:*), Bash(git:*), Bash(git commit:*), Bash(gh pr create:*), Bash(cat:*), Bash(rm:*), Bash(pnpm:*), Bash(npm:*), Bash(yarn:*), Bash(npx:*)
description: Generate PR description and automatically create pull request on GitHub, with automatic code formatting and commit creation
---

## Context

- Current git status: !`git status`
- Changes in this PR: !`git diff main...HEAD`
- Commits in this PR: !`git log --oneline main..HEAD`
- PR template: @.github/pull_request_template.md (if exists)

## Your task

Based on the provided option, perform one of the following actions:

### Options:

- **No option or default**: Format code, create commits, generate PR description and create pull request
- **-p**: Push current branch and create pull request
- **-u**: Update existing pull request description only
- **-c**: Format code and create commits with appropriate granularity (without PR creation)

### Default behavior (no option):

1. Run project formatter (detect from package.json: `format:fix`, `format`, or skip if none)
2. **code-simplifier**: Run Skill tool `code-simplifier:code-simplifier`
3. Split changes into commits with meaningful granularity. Group commits by logical functionality and meaningful content, not just by file. Each commit should represent a single logical change or feature. Use Conventional Commits format: `feat(scope):`, `fix(scope):`, `refactor(scope):`, etc. When creating commits: use the Write tool to write the commit message to `/tmp/commit_msg.txt`, then run `git commit -F /tmp/commit_msg.txt`, then `rm /tmp/commit_msg.txt`.
4. Push current branch to remote repository using `git push -u origin <current-branch>`
5. Create a PR description following the PR template if it exists, otherwise use the format below
6. **Add a Mermaid diagram** that visualizes the changes made in this PR
7. If related issues exist, include `Closes #<issue-number>`
8. Use the Write tool to write the PR body to `/tmp/pr_body.md`, then execute `gh pr create --draft --body-file /tmp/pr_body.md`, then `rm /tmp/pr_body.md`

### With -p option:

1. Push current branch to remote repository using `git push -u origin <current-branch>`
2. Create a PR description following the PR template or default format
3. **Add a Mermaid diagram** that visualizes the changes made in this PR
4. Use the Write tool to write the PR body to `/tmp/pr_body.md`, then execute `gh pr create --draft --body-file /tmp/pr_body.md`, then `rm /tmp/pr_body.md`

### With -u option:

1. Create a PR description following the PR template or default format
2. **Add a Mermaid diagram** that visualizes the changes made in this PR
3. Use the Write tool to write the PR body to `/tmp/pr_body.md`, then update the existing pull request using `gh pr edit --body-file /tmp/pr_body.md`, then `rm /tmp/pr_body.md`

### With -c option:

1. Run project formatter (detect from package.json, skip if none)
2. **code-simplifier**: Run Skill tool `code-simplifier:code-simplifier`
3. Split changes into commits with meaningful granularity. When creating commits: use the Write tool to write the commit message to `/tmp/commit_msg.txt`, then run `git commit -F /tmp/commit_msg.txt`, then `rm /tmp/commit_msg.txt`.

## Default PR Description Format

If no PR template exists, use the following structure:

### WHY
- Reason and context for this change
- Related issues: `Closes #<issue-number>`

### WHAT
- What was implemented
- Key changes in bullet points

### How
- Files and features modified
- Technical approach

### Changes & Flow
- Directory tree of changed files
- Mermaid diagram of the workflow

### Before / After
- Screenshots or description

## Requirements:

1. Follow the template structure exactly if a PR template exists
2. Include specific implementation details
3. List concrete testing steps
4. Always include a Mermaid diagram that shows:
   - Architecture changes (if any)
   - Data flow modifications
   - Component relationships
   - Process flows affected by the changes
5. Be comprehensive but concise
6. **Commit granularity**: Each commit should represent a single logical change, feature, or bug fix. Group related changes together but separate different concerns into different commits. Avoid mixing multiple unrelated changes in a single commit.

### Mermaid Diagram Guidelines:

- Use appropriate diagram types (flowchart, sequence, class, etc.)
- Show before/after states if applicable
- Highlight new or modified components
- Use consistent styling and colors
- Add the diagram in a dedicated section of the PR description

**Generate the PR description and create the pull request automatically.**
