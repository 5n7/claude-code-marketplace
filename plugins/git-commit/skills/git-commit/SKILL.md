---
name: git-commit
description: Commit changes to the git repository
model: haiku
context: fork
---

Create a git commit for staged changes with a well-structured commit message.

## Steps

1. **Check staged changes** by running:
   - `git status` to see what files are staged
   - `git diff --staged` to review the actual changes

2. **Generate a commit message** following Conventional Commits specification.

## Commit Message Format

### Summary line (first line)

- Concise (≤ 72 characters), imperative mood
- Format: `type(scope): description`
- Clearly describe the purpose of the change

### Body (after blank line)

- Use markdown bullet points (`- `) to list significant changes
- Each bullet describes a specific modification, improvement, or addition
- Focus on **what** and **why**, not how
- Avoid repeating the summary line

### Footer (required)

Always include the Claude Code signature:

```
🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

## Example

```
refactor(auth): enhance header auth controls and improve home page UI

- Extract authentication controls into dedicated HeaderAuthControls component
- Add Suspense boundary with loading fallback for auth controls
- Change sign-up from modal to direct link to /sign-up page
- Update home page hero with new brand messaging and taglines
- Add insights dashboard cards section linking to axis-specific pages

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

## Executing the Commit

Use a HEREDOC to preserve formatting:

```bash
git commit -m "$(cat <<'EOF'
type(scope): summary line here

- First change description
- Second change description

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```
