---
name: manus
description: Delegate complex, long-running tasks to Manus AI agent for autonomous execution. Use when user says 'use manus', 'delegate to manus', 'send to manus', 'have manus do', 'ask manus', 'check manus sessions', or when tasks require deep web research, market analysis, product comparisons, stock analysis, competitive research, document generation, data analysis, or multi-step workflows that benefit from autonomous agent execution with parallel processing.
license: Apache-2.0
metadata:
  author: sanjay3290
  version: "1.0"
---

# Manus

Manus is an autonomous AI agent that handles complex tasks asynchronously. Particularly strong at **deep research** with parallel processing, web browsing, and generating comprehensive reports with data visualizations.

## When to Use Manus

- **Deep research** - Market analysis, competitive landscaping, product comparisons
- **Stock/financial analysis** - Company research, technical analysis, price charts
- **Product research** - Feature comparisons across brands, specs analysis
- **Report generation** - Creates markdown reports, CSVs, charts, and visualizations
- **Multi-source synthesis** - Gathers and combines information from multiple websites
- **Long-running tasks** - Anything taking 1-10+ minutes of autonomous work

## Research Prompt Examples

Effective research prompts are specific about scope, sources, and desired output:

**Product comparison:**
```
Find the best 4K monitors for Mac with Thunderbolt connectivity and 120Hz+ refresh rate.
Focus on BenQ, Samsung, Dell, LG. Only include models released in the last year.
Compare specs, prices, and Mac-specific features. Output a comparison table.
```

**Stock/company analysis:**
```
Analyze [TICKER] stock: company profile, recent performance, technical indicators,
valuation metrics, and insider activity. Include a price chart for the past year.
```

**Market research:**
```
Research the [industry] market: key players, market size, growth trends,
recent developments, and competitive landscape. Focus on [region/segment].
```

**Competitive analysis:**
```
Compare [Product A] vs [Product B] vs [Product C]: features, pricing,
user reviews, pros/cons. Create a decision matrix for [use case].
```

## Environment Setup

Requires `MANUS_API_KEY` environment variable. Base URL: `https://api.manus.ai`

## Core Workflow

1. **Create task** → Returns `task_id` immediately
2. **Poll status** → Check until `status` is `completed` or `failed`
3. **Return results** → Extract output text and file attachments

## Creating a Task

```bash
curl -s -X POST "https://api.manus.ai/v1/tasks" -H "API_KEY:$MANUS_API_KEY" -H "Content-Type:application/json" -d '{"prompt":"<task description>","agentProfile":"manus-1.6"}'
```

**Agent Profiles:**
- `manus-1.6-lite` - Fast, simple tasks (quick lookups, simple questions)
- `manus-1.6` - Standard (default, good for most research)
- `manus-1.6-max` - Complex reasoning (deep research, multi-source analysis, detailed reports)

**Response:**
```json
{"task_id":"abc123","task_title":"...","task_url":"https://manus.im/app/abc123"}
```

## Checking Task Status

```bash
curl -s -X GET "https://api.manus.ai/v1/tasks/{task_id}" -H "API_KEY:$MANUS_API_KEY"
```

**Status values:** `pending`, `running`, `completed`, `failed`

Poll every 5-10 seconds until completed.

**Extract text output:**
```bash
curl -s -X GET "https://api.manus.ai/v1/tasks/{task_id}" -H "API_KEY:$MANUS_API_KEY" | jq -r '.output[] | select(.role=="assistant") | .content[] | select(.type=="output_text") | .text'
```

**Extract file attachments:**
```bash
curl -s -X GET "https://api.manus.ai/v1/tasks/{task_id}" -H "API_KEY:$MANUS_API_KEY" | jq -r '.output[] | select(.role=="assistant") | .content[] | select(.type=="output_file") | "\(.fileName): \(.fileUrl)"'
```

## Listing Tasks

```bash
curl -s -X GET "https://api.manus.ai/v1/tasks" -H "API_KEY:$MANUS_API_KEY" | jq '.data[] | {id, status, title: .metadata.task_title}'
```

## Multi-turn Conversations

Continue an existing task by including `taskId`:

```bash
curl -s -X POST "https://api.manus.ai/v1/tasks" -H "API_KEY:$MANUS_API_KEY" -H "Content-Type:application/json" -d '{"prompt":"follow-up question","taskId":"abc123","agentProfile":"manus-1.6"}'
```

## Delete a Task

```bash
curl -s -X DELETE "https://api.manus.ai/v1/tasks/{task_id}" -H "API_KEY:$MANUS_API_KEY"
```

## Advanced Options

| Parameter | Description |
|-----------|-------------|
| `taskMode` | `chat`, `adaptive`, or `agent` |
| `projectId` | Associate with project for shared instructions |
| `attachments` | Array of file objects (see below) |
| `connectors` | Pre-configured connector IDs (Gmail, Calendar, Notion) |
| `createShareableLink` | Enable public access URL |

## File Attachments

Attach files using one of three formats:

**URL attachment:**
```json
{"prompt":"Analyze this","attachments":[{"type":"url","url":"https://example.com/doc.pdf"}]}
```

**Base64 attachment:**
```json
{"prompt":"What's in this image?","attachments":[{"type":"base64","data":"<base64>","mime_type":"image/png"}]}
```

**File ID (after upload):**
```json
{"prompt":"Review this file","attachments":[{"type":"file","file_id":"file-xxx"}]}
```

## Projects

Create a project with shared instructions:
```bash
curl -s -X POST "https://api.manus.ai/v1/projects" -H "API_KEY:$MANUS_API_KEY" -H "Content-Type:application/json" -d '{"name":"My Project","instruction":"Always respond concisely"}'
```

Use project in task:
```bash
curl -s -X POST "https://api.manus.ai/v1/tasks" -H "API_KEY:$MANUS_API_KEY" -H "Content-Type:application/json" -d '{"prompt":"...","projectId":"proj_xxx","agentProfile":"manus-1.6"}'
```

## Best Practices

- Use `manus-1.6-lite` for simple queries (faster, cheaper)
- Use `manus-1.6` or `manus-1.6-max` for research tasks
- Always report `task_url` so user can view progress in browser
- Poll status with reasonable intervals (5-10s for simple, 15-30s for research)
- Check `credit_usage` field to track consumption
- For research: be specific about scope, timeframe, sources, and desired output format
- Research tasks typically produce multiple output files (reports, CSVs, charts) - extract all attachments
- Manus uses parallel processing for multi-faceted research - one prompt can cover multiple angles

## Error Handling

Check for failed tasks:
```bash
curl -s -X GET "https://api.manus.ai/v1/tasks/{task_id}" -H "API_KEY:$MANUS_API_KEY" | jq '{status, error}'
```

If `status` is `failed`, the `error` field contains the reason. Common issues:
- Invalid API key → Check `MANUS_API_KEY` is set
- Task timeout → Simplify prompt or use `manus-1.6-max`
- Rate limited → Wait and retry

## API Reference

For complete endpoint documentation, see [references/api.md](references/api.md).
