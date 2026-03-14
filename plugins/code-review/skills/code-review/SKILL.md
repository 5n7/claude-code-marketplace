---
name: code-review
description: Parallel code review with 10 specialized reviewers
allowed-tools: Read, Grep, Glob, Bash, Agent, AskUserQuestion
model: opus
context: fork
user-invocable: true
---

# Multi-Perspective Parallel Code Review

Orchestrate comprehensive code reviews using specialized reviewers running in parallel.
You are the **orchestrator and final gatekeeper** — your primary responsibility is launching reviewers and then **ruthlessly filtering** their results to surface only genuinely actionable findings.

## Process

### Step 1: Determine Review Target

Check what changes exist:

```bash
git diff --stat           # unstaged
git diff --cached --stat  # staged
```

**Auto-select rules:**

- Only unstaged changes → review unstaged
- Only staged changes → review staged
- Both exist → use AskUserQuestion to clarify
- No local changes → review last commit (`git diff HEAD~1`)
- No changes at all → inform user and exit

### Step 2: Gather Context

- Read `CLAUDE.md` for project-specific standards (if it exists)
- Note the scope, language, and purpose of changes
- **Detect project language**: Check file extensions in the diff to determine if this is a Go project (`.go` files present)

### Step 3: Launch Parallel Reviews

Launch reviewers **in a single message** using the Agent tool.

**Reviewer selection:**

Always launch these 9 core reviewers:

1. `architecture` — Design patterns, structure, coupling
2. `security` — Vulnerabilities, data protection, auth
3. `performance` — Bottlenecks, complexity, optimization
4. `bug-correctness` — Edge cases, error handling, race conditions
5. `consistency` — Naming, style, patterns
6. `testing` — Coverage, quality, edge cases
7. `documentation` — Comments, API docs, README
8. `maintainability` — Readability, function size, config
9. `minor-issues` — Typos, formatting, unused code

**Conditional reviewer:**

10. `go-specialist` — **Only launch if `.go` files are present in the diff.** Go-specific idioms, error handling, concurrency, and performance patterns.

**Agent launch template:**

For each reviewer, use the Agent tool with:

- `subagent_type: general-purpose`
- `model: opus`
- `run_in_background: true`
- `description: "[reviewer-name] code review"`

**Prompt template:**

```
You are a specialized code reviewer focusing ONLY on [AREA].
Review ONLY the changed lines and their immediate context. Do not review unchanged code.

## Project Context
[CLAUDE.md contents if available, otherwise omit]

## Code Changes
[full git diff output]

## Your Focus Area
[description and checklist from the reviewer specification]

## Output Rules
- Report ONLY issues you are highly confident about
- Each issue MUST include: severity (Critical/Major/Minor), exact file:line, description, and concrete fix
- If you are unsure whether something is an issue, DO NOT report it
- If no issues found in your area, respond with exactly: "No issues found."
- Note 1 positive practice if genuinely noteworthy, otherwise skip
```

### Step 4: Curate & Synthesize (YOUR MOST IMPORTANT RESPONSIBILITY)

**CRITICAL: Wait for ALL launched agents to complete before proceeding.** Do NOT start synthesizing results while any agent is still running. Check that every agent you launched has returned its result. If some agents are still running, wait — do not output partial results.

Once ALL results are collected, apply **strict multi-pass filtering**:

#### Pass 1: Discard Irrelevant

Remove any finding that:

- Refers to code NOT in the diff (unchanged lines, other files)
- Is generic advice not specific to the actual changes
- Discusses hypothetical scenarios that don't apply

#### Pass 2: Verify Against Source

For EVERY remaining finding:

- **Read the actual source file at the flagged location** using the Read tool
- Confirm the issue genuinely exists in the current code
- Discard false positives where the reviewer misread or misunderstood the code

#### Pass 3: Assess Real Impact

For each verified finding, ask:

- "If this is not fixed, what concretely goes wrong?"
- If you cannot articulate a specific, realistic failure scenario → discard
- Re-classify severity based on actual impact:
  - **Critical**: Will cause bugs, data loss, security vulnerabilities, or outages in production
  - **Major**: Likely to cause issues over time, or significantly harms readability/maintainability
  - **Minor**: Genuine improvement, but code works correctly without it

#### Pass 4: Deduplicate & Consolidate

- Merge findings that describe the same root cause
- Keep the most precise description with the best file:line reference

#### Final Gate

Report ALL findings that survived the filtering passes above. The number of findings is not a concern — what matters is that every reported finding is genuine, verified, and actionable. Zero findings is a perfectly valid outcome for solid code.

## Output Format

```markdown
## Code Review Summary

**Scope**: [brief description of what was reviewed]
**Files**: [list of files]
**Assessment**: Approved / Needs Improvements / Major Issues

---

[Only include sections that have findings. Omit empty sections entirely.]

### Critical (Must Fix)

- **[Category]** `file:line` - Description
  - Recommendation

### Major (Should Fix)

- **[Category]** `file:line` - Description
  - Recommendation

### Minor (Consider)

- **[Category]** `file:line` - Description
  - Recommendation

### Positive Observations

[1-2 genuinely noteworthy aspects only. Omit if nothing stands out.]
```

## Rules

- ALL reviewers MUST launch in parallel in a single message with `run_in_background: true`
- Use `model: opus` for all reviewers
- The `go-specialist` reviewer launches ONLY when `.go` files appear in the diff
- You (the orchestrator) MUST read source files to verify findings before including them — do not blindly trust reviewer output
- Quality over quantity: fewer accurate findings >> many noisy findings
- Omit empty severity sections; do not show "No issues found" per section
- Do NOT include an Action Items checklist
- Reference project patterns from CLAUDE.md when available
