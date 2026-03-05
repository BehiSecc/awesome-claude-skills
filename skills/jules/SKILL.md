---
name: jules
description: "Delegate coding tasks to Google Jules AI agent for asynchronous execution. Use when user says: 'have Jules fix', 'delegate to Jules', 'send to Jules', 'ask Jules to', 'check Jules sessions', 'pull Jules results', 'jules add tests', 'jules add docs', 'jules review pr'. Handles: bug fixes, documentation, features, tests, refactoring, code reviews. Works with GitHub repos, creates PRs."
license: Apache-2.0
metadata:
  author: sanjay3290
  version: "1.0"
---

# Jules Task Delegation

Delegate coding tasks to Google's Jules AI agent on GitHub repositories.

## Setup (Run Before First Command)

### 1. Install CLI
```bash
which jules || npm install -g @google/jules
```

### 2. Check Auth
```bash
jules remote list --repo
```
If fails → tell user to run `jules login` (or `--no-launch-browser` for headless)

### 3. Auto-Detect Repo
```bash
git remote get-url origin 2>/dev/null | sed -E 's#.*(github\.com)[/:]([^/]+/[^/.]+)(\.git)?#\2#'
```
If not GitHub or not in git repo → ask user for `--repo owner/repo`

### 4. Verify Repo Connected
Check repo is in `jules remote list --repo`. If not → direct to https://jules.google.com

## Commands

### Create Tasks
```bash
jules new "Fix auth bug"                                   # Auto-detected repo
jules new --repo owner/repo "Add unit tests"               # Specific repo
jules new --repo owner/repo --parallel 3 "Implement X"     # Parallel sessions
cat task.md | jules new --repo owner/repo                  # From stdin
```

### Monitor
```bash
jules remote list --session    # All sessions
jules remote list --repo       # Connected repos
```

### Retrieve Results
```bash
jules remote pull --session <id>         # View diff
jules remote pull --session <id> --apply # Apply locally
jules teleport <id>                      # Clone + apply
```

### Latest Session Shortcut
```bash
LATEST=$(jules remote list --session 2>/dev/null | awk 'NR==2 {print $1}')
jules remote pull --session $LATEST
```

## Smart Context Injection

Enrich prompts with current context for better results:

```bash
BRANCH=$(git branch --show-current)
RECENT_FILES=$(git diff --name-only HEAD~3 2>/dev/null | head -10 | tr '\n' ', ')
RECENT_COMMITS=$(git log --oneline -5 | tr '\n' '; ')
STAGED=$(git diff --cached --name-only | tr '\n' ', ')
```

**Use when creating tasks:**
```bash
jules new --repo owner/repo "Fix the bug in auth module. Context: branch=$BRANCH, recently modified: $RECENT_FILES"
```

## Template Prompts

Quick commands for common tasks:

### Add Tests
```bash
FILES=$(git diff --name-only HEAD~3 2>/dev/null | grep -E '\.(js|ts|py|go|java)$' | head -5 | tr '\n' ', ')
jules new "Add unit tests for recently modified files: $FILES. Include edge cases and mocks where needed."
```

### Add Documentation
```bash
FILES=$(git diff --name-only HEAD~3 2>/dev/null | grep -E '\.(js|ts|py|go|java)$' | head -5 | tr '\n' ', ')
jules new "Add documentation comments to: $FILES. Include function descriptions, parameters, return values, and examples."
```

### Fix Lint Errors
```bash
jules new "Fix all linting errors in the codebase. Run the linter, identify issues, and fix them while maintaining code functionality."
```

### Review PR
```bash
PR_NUM=123
PR_INFO=$(gh pr view $PR_NUM --json title,body,files --jq '"\(.title)\n\(.body)\nFiles: \(.files[].path)"')
jules new "Review this PR for bugs, security issues, and improvements: $PR_INFO"
```

## Git Integration (Apply + Commit)

After Jules completes, apply changes to a new branch:

```bash
SESSION_ID="<id>"
TASK_DESC="<brief description>"

# Create branch, apply, commit
git checkout -b "jules/$SESSION_ID"
jules remote pull --session "$SESSION_ID" --apply
git add -A
git commit -m "feat: $TASK_DESC

Jules session: $SESSION_ID"

# Optional: push and create PR
git push -u origin "jules/$SESSION_ID"
gh pr create --title "$TASK_DESC" --body "Automated changes from Jules session $SESSION_ID"
```

## Poll Until Complete

Wait for session to finish:

```bash
SESSION_ID="<id>"
while true; do
  STATUS=$(jules remote list --session 2>/dev/null | grep "$SESSION_ID" | awk '{print $NF}')
  case "$STATUS" in
    Completed)
      echo "Done!"
      jules remote pull --session "$SESSION_ID"
      break ;;
    Failed)
      echo "Failed. Check: https://jules.google.com/session/$SESSION_ID"
      break ;;
    *User*)
      echo "Needs input: https://jules.google.com/session/$SESSION_ID"
      break ;;
    *)
      echo "Status: $STATUS - waiting 30s..."
      sleep 30 ;;
  esac
done
```

## AGENTS.md Template

Create in repo root to improve Jules results:

```markdown
# AGENTS.md

## Project Overview
[Brief description]

## Tech Stack
- Language: [TypeScript/Python/Go/etc.]
- Framework: [React/FastAPI/Gin/etc.]
- Testing: [Jest/pytest/go test/etc.]

## Code Conventions
- [Linter/formatter used]
- [Naming conventions]
- [File organization]

## Testing Requirements
- Unit tests for new features
- Integration tests for APIs
- Coverage target: [X]%

## Build & Deploy
- Build: `[command]`
- Test: `[command]`
```

## Session States

| Status | Action |
|--------|--------|
| Planning / In Progress | Wait |
| Awaiting User F | Respond at web UI |
| Completed | Pull results |
| Failed | Check web UI |

## Notes

- **No CLI reply** → Use web UI for Jules questions
- **No CLI cancel** → Use web UI to cancel
- **GitHub only** → GitLab/Bitbucket not supported
- **AGENTS.md** → Jules reads from repo root for context
