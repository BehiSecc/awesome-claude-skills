---
name: rlm
description: Process large codebases (>100 files) using the Recursive Language Model pattern. Treats code as an external environment, using parallel background agents to map-reduce complex tasks without context rot.
triggers:
  - "analyze codebase"
  - "scan all files"
  - "large repository"
  - "RLM"
  - "find usage of X across the project"
license: MIT
metadata:
  author: ClawFu
  version: 1.0.0
  mcp-server: "@clawfu/mcp-skills"
---

# Recursive Language Model (RLM) Skill

## Core Philosophy
**"Context is an external resource, not a local variable."**

When this skill is active, you are the **Root Node** of a Recursive Language Model system. Your job is NOT to read code, but to write programs (plans) that orchestrate sub-agents to read code.

## Protocol: The RLM Loop

### Phase 1: Choose Your Engine
Decide based on the nature of the data:

| Engine | Use Case | Tool |
|--------|----------|------|
| **Native Mode** | General codebase traversal, finding files, structure. | `find`, `grep`, `bash` |
| **Strict Mode** | Dense data analysis (logs, CSVs, massive single files). | `python3 ~/.claude/skills/rlm/rlm.py` |

### Phase 2: Index & Filter (The "Peeking" Phase)
**Goal**: Identify relevant data without loading it.
1.  **Native**: Use `find` or `grep -l`.
2.  **Strict**: Use `python3 .../rlm.py peek "query"`.
    *   *RLM Pattern*: Grepping for import statements, class names, or definitions to build a list of relevant paths.

### Phase 3: Parallel Map (The "Sub-Query" Phase)
**Goal**: Process chunks in parallel using fresh contexts.
1.  **Divide**: Split the work into atomic units.
    - **Strict Mode**: `python3 .../rlm.py chunk --pattern "*.log"` -> Returns JSON chunks.
2.  **Spawn**: Use `background_task` to launch parallel agents.
    *   *Constraint*: Launch at least 3-5 agents in parallel for broad tasks.
    *   *Prompting*: Give each background agent ONE specific chunk or file path.
    *   *Format*: `background_task(agent="explore", prompt="Analyze chunk #5 of big.log: {content}...")`

### Phase 4: Reduce & Synthesize (The "Aggregation" Phase)
**Goal**: Combine results into a coherent answer.
1.  **Collect**: Read the outputs from `background_task` (via `background_output`).
2.  **Synthesize**: Look for patterns, consensus, or specific answers in the aggregated data.
3.  **Refine**: If the answer is incomplete, perform a second RLM recursion on the specific missing pieces.

## Critical Instructions

1.  **NEVER** use `cat *` or read more than 3-5 files into your main context at once.
2.  **ALWAYS** prefer `background_task` for reading/analyzing file contents when the file count > 1.
3.  **Use `rlm.py`** for programmatic slicing of large files that `grep` can't handle well.
4.  **Python is your Memory**: If you need to track state across 50 files, write a Python script (or use `rlm.py`) to scan them and output a summary.

## Example Workflow: "Find all API endpoints and check for Auth"

**Wrong Way (Monolithic)**:
- `read src/api/routes.ts`
- `read src/api/users.ts`
- ... (Context fills up, reasoning degrades)

**RLM Way (Recursive)**:
1.  **Filter**: `grep -l "@Controller" src/**/*.ts` -> Returns 20 files.
2.  **Map**: 
    - `background_task(prompt="Read src/api/routes.ts. Extract all endpoints and their @Auth decorators.")`
    - `background_task(prompt="Read src/api/users.ts. Extract all endpoints and their @Auth decorators.")`
    - ... (Launch all 20)
3.  **Reduce**: 
    - Collect all 20 outputs.
    - Compile into a single table.
    - Identify missing auth.

## Recovery Mode
If `background_task` is unavailable or fails:
1.  Fall back to **Iterative Python Scripting**.
2.  Write a Python script that loads each file, runs a regex/AST check, and prints the result to stdout.
3.  Read the script's stdout.

---

## What Claude Does vs What You Decide

| Claude handles | You provide |
|---------------|-------------|
| Orchestrating parallel agents | Initial query and success criteria |
| Chunking large files for processing | Judgment on result quality |
| Synthesizing results from subagents | Final interpretation and action |
| Writing filtering scripts | Validation of completeness |
| Managing context isolation | Decision on when to stop recursing |

---

## Skill Boundaries

### This skill excels for:
- Codebases with >100 files
- Finding patterns across many files
- Audit tasks (security, auth, logging)
- Large file analysis (logs, data dumps)

### This skill is NOT ideal for:
- Small projects (<50 files) → Direct reading faster
- Single file analysis → Overkill
- Tasks requiring file modification → Use different approach

---

## Skill Metadata

```yaml
name: rlm
category: meta
version: 2.0
author: GUIA
source_expert: Recursive Language Model pattern
difficulty: advanced
mode: cyborg
tags: [rlm, large-codebase, parallel-agents, map-reduce, context-management]
created: 2026-02-03
updated: 2026-02-03
```
