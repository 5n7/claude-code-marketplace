---
name: code-review
description: Parallel code review with 9 specialized reviewers
allowed-tools: Read, Grep, Glob, Bash, Task, TaskOutput, AskUserQuestion
model: sonnet
context: fork
user-invocable: true
---

# Multi-Perspective Parallel Code Review

Orchestrate comprehensive code reviews using 9 specialized reviewers running in parallel.

## Process

### Step 1: Determine Review Target

First, check what changes exist:

```bash
git diff --stat           # unstaged
git diff --cached --stat  # staged
```

**If changes are ambiguous or unclear**, use AskUserQuestion to clarify:

```
Question: "What would you like to review?"
Options:
- "Unstaged changes" (git diff)
- "Staged changes" (git diff --cached)
- "Last commit" (git diff HEAD~1)
- "Changes from main branch" (git diff main...HEAD)
```

**Auto-select** if only one type of change exists:

- Only unstaged → review unstaged
- Only staged → review staged
- No local changes → review last commit

If no changes found anywhere, inform the user and exit.

### Step 2: Gather Context

- Read `CLAUDE.md` for project-specific standards
- Note the scope and purpose of changes

### Step 3: Launch Parallel Reviews

Launch ALL 9 reviewers **in a single message** using Task tool with `run_in_background: true` and `model: sonnet`.

See `reviewers.md` for each reviewer's focus area and checklist.

**Prompt template for each reviewer:**

```
You are a specialized code reviewer focusing ONLY on [AREA].

## Code Changes
[git diff output]

## Review Checklist
[checklist from reviewers.md]

## Output Format
For each issue found:
- [Severity: Critical/Major/Minor] `file:line` - Description
  - Recommendation: How to fix

If no issues found, state "No [AREA] issues found."
Also note any positive practices observed.
```

### Step 4: Collect & Synthesize

Use `TaskOutput` to collect all results, then:

1. Deduplicate overlapping findings
2. Sort by severity (Critical > Major > Minor)
3. Group related issues

## Output Format

```markdown
## Code Review Summary

**Files Reviewed**: [files]
**Overall Assessment**: ✅ Approved / ⚠️ Needs Improvements / ❌ Major Issues

---

### 🚨 Critical (Must Fix)

- **[Category]** `file:line` - Description
  - 💡 Recommendation

### ⚠️ Major (Should Fix)

- **[Category]** `file:line` - Description
  - 💡 Recommendation

### 📝 Minor (Consider)

- **[Category]** `file:line` - Description
  - 💡 Recommendation

### ✨ Positive Observations

[Well-implemented aspects]

### 📋 Action Items

- [ ] 🚨 Critical: [item]
- [ ] ⚠️ Major: [item]
- [ ] 📝 Minor: [item]
```

## Guidelines

- All 9 reviewers MUST run in parallel with `run_in_background: true`
- Use `model: sonnet` for reviewers (cost-efficient)
- Each reviewer focuses ONLY on their specialty
- Reference project patterns from CLAUDE.md when available
- Be constructive - highlight good practices alongside issues
