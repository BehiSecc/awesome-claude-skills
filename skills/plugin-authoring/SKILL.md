---
name: plugin-authoring
description: Use when creating, modifying, or debugging Claude Code plugins. Triggers on .claude-plugin/, plugin.json, marketplace.json, commands/, agents/, skills/, hooks/ directories. Provides schemas, templates, validation workflows, and troubleshooting.
allowed-tools: Read, Grep, Glob
---

# Plugin Authoring (Skill)

You are the canonical guide for Claude Code plugin development. Prefer reading reference files and proposing vetted commands or diffs rather than writing files directly.

**Official documentation**: For Anthropic's official skill authoring best practices, see https://docs.anthropic.com/en/docs/agents-and-tools/agent-skills/skill-authoring-best-practices

## Triggers & Scope

Activate whenever context includes `.claude-plugin/`, `plugin.json`, `marketplace.json`, `commands/`, `agents/`, `skills/`, or `hooks/`.

## Flow of Operation

1. **Diagnose** current repo layout (read-only)
2. **Propose** the minimal safe action (scaffold, validate, or review)
3. **Execute** via `/plugin-development:...` commands when the user agrees
4. **Escalate** to the **plugin-reviewer** agent for deep audits
5. **Guardrails**: default to read-only; ask before edits

## Quick Links (Progressive Disclosure)

- **Schemas**: [schemas/plugin-manifest.md](schemas/plugin-manifest.md), [schemas/hooks-schema.md](schemas/hooks-schema.md), [schemas/marketplace-schema.md](schemas/marketplace-schema.md)
- **Templates**: [templates/](templates/)
- **Examples**: [examples/](examples/)
- **Best practices**: [best-practices/](best-practices/)
- **Common mistakes**: [best-practices/common-mistakes.md](best-practices/common-mistakes.md)
- **Testing this skill**: [testing-plugin-authoring.md](testing-plugin-authoring.md)

## Checklists

### Component Checklist

```
□ .claude-plugin/plugin.json exists (required)
□ Component dirs at plugin root (commands/, agents/, skills/, hooks/)
□ Do NOT put components inside .claude-plugin/ directory
□ Commands use kebab-case naming
□ Skills have valid frontmatter (name + description required, optional: model, allowed-tools)
□ Skills name validation:
  - Matches directory name
  - Lowercase letters, numbers, hyphens only
  - Max 64 characters
  - No reserved words ('anthropic', 'claude')
  - No XML tags
□ Hooks use ${CLAUDE_PLUGIN_ROOT} for paths (not relative paths)
□ All scripts are executable (chmod +x)
```

### Release Checklist

```
□ plugin.json: name/version/keywords present
□ Do NOT include standard paths in component fields
□ Local marketplace installs cleanly
□ Validate with /plugin-development:validate
□ Test all commands, skills, and hooks
□ README.md exists with usage examples
```

## Red Flags (STOP If You're About To...)

- Put `commands/`, `agents/`, `skills/`, or `hooks/` inside `.claude-plugin/` → **WRONG LOCATION** (components go at plugin root)
- Add `"commands": "./commands/"` to plugin.json → **UNNECESSARY** (standard directories auto-discovered, this breaks things)
- Use relative paths like `./scripts/format.sh` in hooks → **USE** `${CLAUDE_PLUGIN_ROOT}/scripts/format.sh`
- Skip `/plugin-development:validate` before testing → **ALWAYS VALIDATE FIRST**
- Forget `chmod +x` on hook scripts → **Scripts won't execute (silent failure)**
- Use 'claude' or 'anthropic' in skill names → **RESERVED WORDS (will be rejected)**

**All of these cause silent failures. When in doubt, validate.**

For detailed explanations: [best-practices/common-mistakes.md](best-practices/common-mistakes.md)

## Why Validation Matters

| Skip This | What Happens |
|-----------|--------------|
| Validate manifest | Plugin won't load, no error message |
| chmod +x scripts | Hooks silently fail |
| ${CLAUDE_PLUGIN_ROOT} | Works in dev, breaks on install |
| Standard directory rules | Components not discovered |

**Running `/plugin-development:validate` catches 90% of issues before debugging starts.**

## Playbooks

- **Scaffold** → `/plugin-development:init <name>` then fill templates
- **Add a component** → `/plugin-development:add-command|add-skill|add-agent|add-hook`
- **Validate** → `/plugin-development:validate` (schema & structure checks)
- **Test locally** → `/plugin-development:test-local` (dev marketplace)

## Common Workflows

### Creating a New Plugin

1. Run `/plugin-development:init <plugin-name>` to scaffold structure
2. Edit `.claude-plugin/plugin.json` with your metadata
3. Add components using `/plugin-development:add-command`, etc.
4. Validate with `/plugin-development:validate`
5. Test locally with `/plugin-development:test-local`

### Adding a Slash Command

1. Run `/plugin-development:add-command <name> <description>`
2. Edit `commands/<name>.md` with instructions
3. Add frontmatter: `description` and `argument-hint`
4. Test: `/plugin install` your plugin, then `/<name>`

### Adding a Skill

1. Run `/plugin-development:add-skill <name> <when-to-use>`
2. Edit `skills/<name>/SKILL.md` with your instructions
3. **Frontmatter requirements**:
   - `name`: lowercase letters, numbers, and hyphens only, max 64 chars (required). Cannot contain reserved words 'anthropic' or 'claude'. Cannot contain XML tags.
   - `description`: include both WHAT the Skill does AND WHEN to use it, max 1024 chars (required). Cannot contain XML tags.
   - `model`: specify which Claude model to use, e.g., `model: claude-sonnet-4-20250514` (optional, defaults to conversation's model)
   - `allowed-tools`: comma-separated list of tools (optional). Tools listed don't require permission to use when Skill is active. If omitted, Skill doesn't restrict tools.
4. Keep SKILL.md under 500 lines for optimal performance; place details in sibling files (reference.md, examples.md, scripts/)

### Troubleshooting

- **Plugin not loading?** Check `plugin.json` paths are relative to plugin root. Do NOT include `commands`, `agents`, `skills`, or `hooks` fields for standard directories.
- **Commands not showing?** Verify `commands/` directory exists at plugin root with `.md` files. Do NOT add `commands` field to `plugin.json` for standard paths.
- **Hooks not running?** Ensure scripts are executable (`chmod +x`) and use `${CLAUDE_PLUGIN_ROOT}` for paths
- **Skill not triggering?** Check `name` matches directory and uses lowercase letters, numbers, and hyphens only (max 64 chars). Ensure `description` includes both what and when to use (max 1024 chars). Neither field can contain XML tags.

## Notes

- Prefer templates & scripts over freeform generation for deterministic tasks
- If writes are needed, propose a command or a PR-style diff first
- For complex audits, delegate to `/agents plugin-reviewer`
- Always validate with `/plugin-development:validate` before testing
