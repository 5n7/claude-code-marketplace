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

### Step 4: Curate & Synthesize

Use `TaskOutput` to collect all reviewer results.

**Critical: You are the final gatekeeper. Do NOT simply aggregate all findings. Critically evaluate each issue and discard noise.**

Apply the following filters to every finding:

1. **Relevance filter**: Does this issue actually apply to the changed code? Discard generic advice that doesn't relate to the specific diff.
2. **Actionability filter**: Is this a concrete, fixable issue with a clear location (`file:line`)? Discard vague or aspirational suggestions (e.g., "consider adding more tests").
3. **Impact filter**: Would fixing this issue meaningfully improve the code's correctness, security, or maintainability? Discard nitpicks and stylistic preferences that don't affect quality.
4. **Duplication filter**: If multiple reviewers flagged the same underlying issue, consolidate into a single finding.
5. **False positive filter**: Re-read the actual code at the flagged location. If the reviewer misunderstood the code or the issue doesn't exist, discard it.

**Severity re-assessment**: After filtering, re-evaluate severity based on actual impact:

- **Critical**: Will cause bugs, data loss, security vulnerabilities, or outages in production
- **Major**: Likely to cause issues over time, or significantly harms readability/maintainability
- **Minor**: Genuine improvements, but code works correctly without them

**Target output**: Aim for **3-7 high-signal findings** total. It is perfectly acceptable to report zero issues if the code is solid. A review with 0-2 findings is better than one with 15 low-value items.

## Output Format

```markdown
## Code Review Summary

**Files Reviewed**: [files]
**Overall Assessment**: ✅ Approved / ⚠️ Needs Improvements / ❌ Major Issues

---

[Only include severity sections that have findings. Omit empty sections.]

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

[1-2 genuinely noteworthy aspects. Skip if nothing stands out.]
```

## Guidelines

- All 9 reviewers MUST run in parallel with `run_in_background: true`
- Use `model: sonnet` for reviewers (cost-efficient)
- Each reviewer focuses ONLY on their specialty
- The main agent (you) is responsible for **filtering out noise** — quality over quantity
- Omit empty severity sections entirely; do not show "No issues found" per section
- Do NOT include an Action Items checklist — the findings themselves are the action items
- Reference project patterns from CLAUDE.md when available
