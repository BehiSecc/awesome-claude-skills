---
name: terminal-title
description: Automatically updates terminal window title to reflect the current high-level task. Use at the start of every Claude Code session when the user provides their first prompt, and whenever the user switches to a distinctly new high-level task. Helps developers manage multiple Claude Code terminals by providing clear, at-a-glance identification of what each terminal is working on.
min_claude_code_version: "1.0.0"
version: "1.1.0"
---

# Terminal Title

## Overview

Automatically sets descriptive terminal window titles based on the task Claude is working on. Essential for developers running multiple Claude Code instances who need to quickly identify which terminal is handling which task.

## When to Use

**Always trigger this skill:**
- At the start of every new Claude Code session (after receiving the first user prompt)
- When switching to a substantially different task (e.g., from "API Integration" to "Database Migration")

**Trigger on task switches like these:**
- Switching from frontend work to backend work
- Moving from debugging to new feature development
- Changing from one module/component to a completely different one
- Starting work on a different part of the system (e.g., from auth to payments)

**Do NOT trigger for:**
- Follow-up questions about the same task ("Can you add a comment to that function?")
- Small refinements to current work ("Make it blue instead of red")
- Debugging the same feature you just built
- Clarifications ("What did you mean by X?")
- Iterating on the same component or module
- Mid-task status updates or progress checks

## How It Works

1. **Extract Task Summary**: Analyze the user's prompt to identify the high-level task
2. **Generate Title**: Create a concise, descriptive title (max 40 characters)
3. **Set Title**: Execute the `scripts/set_title.sh` script with the generated title
4. **No Confirmation Needed**: This happens automatically in the background

## Title Format Guidelines

**Good titles:**
- "API Integration: Auth Flow"
- "Fix: Login Bug"
- "DB Migration: Users Table"
- "Build: Dashboard UI"
- "Refactor: Payment Module"

**Displayed as (with automatic folder prefix):**
- `my-project | API Integration: Auth Flow`
- `my-project | Fix: Login Bug`

**Bad titles:**
- Too long: "Implementing the new authentication system with OAuth2.0 support" (exceeds 40 chars)
- Too generic: "Working" or "Coding"
- Too verbose: "The user wants me to help them with..."

**Format pattern:**
```
[Action/Category]: [Specific Focus]
```

The script automatically prefixes titles with the current directory name (usually the repo name) for easy identification across multiple terminals.

Keep titles concise, actionable, and immediately recognizable.

## Common Mistakes to Avoid

**❌ Too Verbose:**
- Bad: "Working on implementing the user authentication system with JWT tokens"
- Good: "Build: JWT Auth"

**❌ Too Vague:**
- Bad: "Code stuff"
- Bad: "Working"
- Good: "Refactor: API Layer"

**❌ Including System Information:**
- Bad: "john-macbook-pro: Debug app"
- Bad: "/Users/john/project: Build feature"
- Good: "Debug: App Issues"

**❌ Using Complete Sentences:**
- Bad: "I am working on the dashboard component"
- Good: "Build: Dashboard UI"

## Implementation

**Execute the title script:**
```bash
bash scripts/set_title.sh "Your Title Here"
```

**Example workflow:**
```bash
# User asks: "Help me debug the authentication flow in the API"
bash scripts/set_title.sh "Debug: Auth API Flow"

# User asks: "Create a React component for the user profile page"
bash scripts/set_title.sh "Build: User Profile UI"

# User asks: "Write tests for the payment processing module"
bash scripts/set_title.sh "Test: Payment Module"
```

## Script Details

The `scripts/set_title.sh` script uses ANSI escape sequences to set the terminal title. It's compatible with:
- macOS Terminal
- iTerm2
- Alacritty
- Most modern terminal emulators (xterm, rxvt, screen, tmux)

The script accepts a single argument (the title string) and exits silently if no title is provided (fail-safe behavior).

## Automatic Directory Prefix

The script automatically prefixes all titles with the current directory name (usually the repo/project name). This makes it easy to identify which project each terminal is working on:

```
my-project | Build: Dashboard UI
another-repo | Debug: Auth API
```

## Optional Custom Prefix

Users can optionally add an additional custom prefix by setting the `CLAUDE_TITLE_PREFIX` environment variable:

```bash
export CLAUDE_TITLE_PREFIX="🤖"
```

This produces titles like: `🤖 my-project | Build: Dashboard UI`

**Note:** You don't need to check for these variables or modify your behavior. The script handles this automatically.
