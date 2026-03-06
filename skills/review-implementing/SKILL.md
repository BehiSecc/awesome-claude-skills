---
name: review-implementing
description: Process and implement code review feedback systematically. Use when user provides reviewer comments, PR feedback, code review notes, or asks to implement suggestions from reviews.
---

# Review Feedback Implementation

Systematically process and implement changes based on code review feedback.

## When to Use

- Provides reviewer comments or feedback
- Pastes PR review notes
- Mentions implementing review suggestions
- Says "address these comments" or "implement feedback"
- Shares list of changes requested by reviewers

## Systematic Workflow

### 1. Parse Reviewer Notes

Identify individual feedback items:
- Split numbered lists (1., 2., etc.)
- Handle bullet points or unnumbered feedback
- Extract distinct change requests
- Clarify ambiguous items before starting

### 2. Create Todo List

Use TodoWrite tool to create actionable tasks:
- Each feedback item becomes one or more todos
- Break down complex feedback into smaller tasks
- Make tasks specific and measurable
- Mark first task as `in_progress` before starting

Example:
```
- Add type hints to extract function
- Fix duplicate tag detection logic
- Update docstring in chain.py
- Add unit test for edge case
```

### 3. Implement Changes Systematically

For each todo item:

**Locate relevant code:**
- Use Grep to search for functions/classes
- Use Glob to find files by pattern
- Read current implementation

**Make changes:**
- Use Edit tool for modifications
- Follow project conventions (CLAUDE.md)
- Preserve existing functionality unless changing behavior

**Verify changes:**
- Check syntax correctness
- Run relevant tests if applicable
- Ensure changes address reviewer's intent

**Update status:**
- Mark todo as `completed` immediately after finishing
- Move to next todo (only one `in_progress` at a time)

### 4. Handle Different Feedback Types

**Code changes:**
- Use Edit tool for existing code
- Follow type hint conventions (PEP 604/585)
- Maintain consistent style

**New features:**
- Create new files with Write tool if needed
- Add corresponding tests
- Update documentation

**Documentation:**
- Update docstrings following project style
- Modify markdown files as needed
- Keep explanations concise

**Tests:**
- Write tests as functions, not classes
- Use descriptive names
- Follow pytest conventions

**Refactoring:**
- Preserve functionality
- Improve code structure
- Run tests to verify no regressions

### 5. Validation

After implementing changes:
- Run affected tests
- Check for linting errors: `uv run ruff check`
- Verify changes don't break existing functionality

### 6. Communication

Keep user informed:
- Update todo list in real-time
- Ask for clarification on ambiguous feedback
- Report blockers or challenges
- Summarize changes at completion

## Edge Cases

**Conflicting feedback:**
- Ask user for guidance
- Explain conflict clearly

**Breaking changes required:**
- Notify user before implementing
- Discuss impact and alternatives

**Tests fail after changes:**
- Fix tests before marking todo complete
- Ensure all related tests pass

**Referenced code doesn't exist:**
- Ask user for clarification
- Verify understanding before proceeding

## Important Guidelines

- **Always use TodoWrite** for tracking progress
- **Mark todos completed immediately** after each item
- **Only one todo in_progress** at any time
- **Don't batch completions** - update status in real-time
- **Ask questions** for unclear feedback
- **Run tests** if changes affect tested code
- **Follow CLAUDE.md conventions** for all code changes
- **Use conventional commits** if creating commits afterward
