---
name: agnix
description: "Use when user asks to 'lint agent configs', 'validate skills', 'check CLAUDE.md', 'validate hooks', 'lint MCP'. Validates agent configuration files against 230 rules."
allowed-tools: Bash(agnix:*), Bash(cargo:*), Read, Glob, Grep
---

# agnix

Lint agent configurations before they break your workflow. Validates Skills, Hooks, MCP, Memory, Plugins across Claude Code, Cursor, GitHub Copilot, and Codex CLI.

## When to Use

Invoke when user asks to:
- "Lint my agent configs"
- "Validate my skills"
- "Check my CLAUDE.md"
- "Validate hooks"
- "Lint MCP configs"
- "Fix agent configuration issues"

## Supported Files

| File Type | Examples |
|-----------|----------|
| Skills | `SKILL.md` |
| Memory | `CLAUDE.md`, `AGENTS.md` |
| Hooks | `.claude/settings.json` |
| MCP | `*.mcp.json` |
| Cursor | `.cursor/rules/*.mdc` |
| Copilot | `.github/copilot-instructions.md` |

## Execution

### 1. Check if agnix is installed

```bash
agnix --version
```

If not found, install:
```bash
cargo install agnix-cli
```

### 2. Validate

```bash
agnix .
```

### 3. If issues found and --fix requested

```bash
agnix --fix .
```

### 4. Re-validate to confirm

```bash
agnix .
```

## CLI Reference

| Command | Description |
|---------|-------------|
| `agnix .` | Validate current project |
| `agnix --fix .` | Auto-fix issues |
| `agnix --strict .` | Treat warnings as errors |
| `agnix --target claude-code .` | Only Claude Code rules |
| `agnix --target cursor .` | Only Cursor rules |
| `agnix --watch .` | Watch mode |
| `agnix --format json .` | JSON output |

## Output Format

```
CLAUDE.md:15:1 warning: Generic instruction 'Be helpful' [fixable]
  help: Remove generic instructions. Claude already knows this.

skills/review/SKILL.md:3:1 error: Invalid name [fixable]
  help: Use lowercase letters and hyphens only

Found 1 error, 1 warning (2 fixable)
```

## Common Issues & Fixes

| Issue | Solution |
|-------|----------|
| Invalid skill name | Use lowercase with hyphens: `my-skill` |
| Generic instructions | Remove "be helpful", "be accurate" |
| Missing trigger phrase | Add "Use when..." to description |
| Directory/name mismatch | Rename directory to match `name:` field |

## Links

- [GitHub](https://github.com/avifenesh/agnix)
- [Rules Reference](https://avifenesh.github.io/agnix/docs/rules)
