---
name: linear
description: Work with Linear issues via CLI - use this skill whenever the user asks about Linear issues, creating, updating, commenting on, or deleting issues, or checking issue status and details
version: 0.1.0
---

# Linear Issue Management

**Use this skill whenever the user mentions Linear or asks to work with issues.**

Lightweight CLI to interact with Linear's issue tracking system. All commands run from the skill directory using `./linear`.

## Setup

Dependencies install automatically on first run. API key errors are self-explanatory.

## Command Pattern

```bash
./linear <resource> <action> [arguments] [options]
```

Resources: `issue`, `user`, `team`, `project`

## Commands

### List Users
```bash
./linear user list
```
Returns: `#<user-id>	<name>	<email>`

### List Teams
```bash
./linear team list
```
Returns: `#<team-id>	<name>	<key>`

### List Projects
```bash
./linear project list
```
Returns: `#<project-id>	<name>	<state>`

### List Issues
```bash
./linear issue list [options]
```
**Options:**
- `--team <id>` - Filter by team ID
- `--assignee <id>` - Filter by user ID
- `--status <name>` - Filter by status name (case-sensitive)
- `--limit <n>` - Limit results (default: 50)

Returns: `#<identifier>	<title>	<status>	<assignee>`

**Examples:**
```bash
./linear issue list --team abc123 --limit 10
./linear issue list --assignee def456 --status "In Progress"
```

### View Issue
```bash
./linear issue view <id-or-key>
```
**Arguments:**
- `<id-or-key>` - Issue identifier (e.g., `ENG-123`) or UUID

Returns full issue details including title, status, assignee, team, priority, labels, dates, description, and comments.

### Create Issue
```bash
./linear issue create <title> [options]
```
**Arguments:**
- `<title>` - Issue title (multi-word titles auto-combined)

**Options:**
- `--team <id>` - Team ID (required)
- `--description <text>` - Issue description
- `--assignee <id>` - User ID
- `--priority <n>` - Priority (0=None, 1=Urgent, 2=High, 3=Medium, 4=Low)
- `--status <name>` - Initial status

**Example:**
```bash
./linear issue create "Fix login bug" --team abc123 --priority 2
```

### Add Comment
```bash
./linear issue comment <id-or-key> <text>
```
Multi-word text auto-combined. No quotes needed.

### Update Issue
```bash
./linear issue update <id-or-key> [options]
```
**Options:**
- `--status <name>` - Update status
- `--assignee <id>` - Update assignee
- `--priority <n>` - Update priority
- `--title <text>` - Update title
- `--description <text>` - Update description

Can update multiple fields in one command.

**Example:**
```bash
./linear issue update ENG-123 --status "In Progress" --assignee abc123
```

### Delete Issue
```bash
./linear issue delete <id-or-key>
```
Soft delete (moves to trash, recoverable).

## Important Notes

- Issue identifiers are case-insensitive (`ENG-123` = `eng-123`)
- Status names are case-sensitive ("In Progress" ≠ "in progress")
- User/team IDs are UUIDs (get from list commands)
- Issue keys format: `<TEAM_KEY>-<NUMBER>` (e.g., ENG-123)
- All commands support `--json` flag for machine-readable output
- Use `--help` on any command for details
