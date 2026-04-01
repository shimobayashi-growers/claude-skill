---
name: implement-to-pr
description: |
  Complete workflow from implementation task to PR creation. Executes planning, branch creation, implementation, user confirmation, commits, and PR creation sequentially.

  Trigger conditions:
  - Explicit: "Create PR", "Implement and make PR", "Build X and create pull request"
  - Auto-detect: "Implement X", "Create X feature", "Add X", "Fix X" - implementation requests requiring multiple steps

  Target tasks: New feature implementation, bug fixes, refactoring, component additions
---

# Implement to PR Workflow

## Workflow Overview

```
1. Requirements (Issue/Description)
   ↓
2. Planning (ultrathink in plan mode)
   ↓
3. Branch Creation
   ↓
4. Implementation
   ↓
5. User Confirmation (before commit)
   ↓
6. Granular Commits
   ↓
7. Error Fixing & PR Creation
```

## Step 1: Requirements Clarification

- Read and understand the issue or user description
- If requirements are unclear, use `AskUserQuestion` to clarify
- Identify scope: files affected, dependencies, breaking changes

## Step 2: Planning with Ultrathink

### Enter Plan Mode
1. Use `EnterPlanMode` tool
2. Investigate codebase using Glob, Grep, Read
3. Write implementation plan to plan file with detailed analysis
4. Use ultrathink for complex architectural decisions
5. Get user approval, then `ExitPlanMode`

### Skip Planning When
- Single file, trivial change
- User explicitly says "no planning needed"
- Clear, specific instructions already provided

## Step 3: Branch Creation

**IMPORTANT: Always create and switch to a new branch before implementation**

```bash
# Check current status
git status
git branch

# Create and switch to feature branch
git checkout -b feature/<issue-number>-<short-description>
# or
git checkout -b fix/<issue-number>-<short-description>
```

Branch naming convention:
- `feature/<issue>-<desc>` for new features
- `fix/<issue>-<desc>` for bug fixes
- `refactor/<issue>-<desc>` for refactoring

## Step 4: Implementation

1. Create task list using `TodoWrite`
2. Execute each task sequentially
3. Update task status immediately upon completion
4. Follow existing code patterns
5. Make minimal necessary changes

## Step 5: User Confirmation Before Commit

**CRITICAL: Always confirm with user before committing**

After implementation is complete:
1. Run tests/build to verify changes work
   ```bash
   pnpm build
   pnpm lint
   ```
2. Show summary of changes to user
3. Ask for confirmation or feedback using `AskUserQuestion`
4. Wait for user approval before proceeding to commit

### If User Requests Changes
- Implement requested modifications
- Re-run tests
- Ask for confirmation again

## Step 6: Granular Commits

**Only proceed after user confirmation**

### Commit Strategy
- Create meaningful, atomic commits
- One commit = one logical change
- Group related changes together

### Commit Message Format
```
<type>(<scope>): <subject>

<body>

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <model>
```

**Types**: feat, fix, refactor, chore, docs, style, test

### Commit Commands
```bash
git add <specific-files>
git commit -m "<message>"
```

## Step 7: Error Fixing & PR Creation

### Fix Any Errors
If build/lint errors occur:
1. Analyze error message
2. Fix the issue
3. Re-run verification
4. Repeat until all errors resolved

### PR Creation
1. Read `.github/pull_request_template.md` if exists
2. Follow project template format
3. If no template, use standard format:

```markdown
## Summary
- Change 1
- Change 2

## Test plan
- [ ] Test item 1
- [ ] Test item 2

🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

### Commands
```bash
git push -u origin <branch-name>
gh pr create --title "<title>" --body "<body>"
```

Report PR URL to user upon completion.

## Decision Guidelines

### Use AskUserQuestion When
- Multiple valid implementation approaches exist
- Requirements are ambiguous
- Changes are breaking/destructive
- User intent is unclear
- Before committing (mandatory)

### Proceed Autonomously When
- Clear, specific instructions given
- Following established patterns
- Minor, safe changes
- User has pre-approved approach
