---
name: charles-proxy-extract
description: Extracts HTTP/HTTPS request and response data from Charles Proxy session files (.chlsj format), including URLs, methods, status codes, headers, request bodies, and response bodies. Use when analyzing captured network traffic from Charles Proxy debug sessions, inspecting API calls, debugging HTTP requests, or examining proxy logs.
allowed-tools: Bash
---

# Charles Proxy Session Extractor

Parses and extracts structured data from Charles Proxy session files (.chlsj format).

## Prerequisites

- Python 3.x (no external dependencies required)
- Charles Proxy session file in .chlsj format

## When to Use This Skill

Use this skill when the user:
- Mentions "Charles Proxy" or "Charles session"
- Asks to "extract", "analyze", or "inspect" .chlsj files
- Wants to filter HTTP/HTTPS requests by endpoint or method
- Needs to examine API request/response data from proxy logs
- Wants to export network traffic data to JSON

## How to Execute This Skill

When the user asks to extract, analyze, or inspect Charles Proxy session files, run the Python script using the Bash tool:

```bash
python3 ./extract_responses.py <file.chlsj> <pattern> [options]
```

### Required Parameters
1. `<file.chlsj>` - Path to the Charles Proxy session file (use exact path provided by user)
2. `<pattern>` - URL path pattern to match (e.g., "/today", "/logs", "/" for all)

### Optional Flags
- `-m, --method METHOD` - Filter by HTTP method (GET, POST, PUT, PATCH, DELETE)
- `-f, --first-only` - Show only first matching request (for quick inspection)
- `-s, --summary-only` - Show statistics without response bodies
- `-o, --output FILE` - Save responses to JSON file
- `--no-pretty` - Disable JSON pretty-printing

### Execution Examples

**Extract all /today responses:**
```bash
python3 ./extract_responses.py session.chlsj "/today"
```

**Filter by POST method (automatically shows request bodies):**
```bash
python3 ./extract_responses.py session.chlsj "/logs" --method POST
```

**Quick peek (first result only):**
```bash
python3 ./extract_responses.py session.chlsj "/users" --first-only
```

**Summary without bodies:**
```bash
python3 ./extract_responses.py session.chlsj "/" --summary-only
```

**Export to file:**
```bash
python3 ./extract_responses.py session.chlsj "/items" --output items_data.json
```

### User Request Patterns

When users say things like:
- "Extract [endpoint] from [file]" → Use basic extraction with pattern matching
- "Show POST/PUT/PATCH to [endpoint]" → Add `--method` flag (request bodies auto-shown)
- "First [endpoint] response" → Add `--first-only` flag
- "Summarize [file]" or "What's in [file]" → Add `--summary-only` flag
- "Save [endpoint] to [output]" → Add `--output` flag
- "Compare [endpoint] with model" → Extract first response, then analyze structure

### Important Notes
- Pattern matching is case-sensitive substring matching
- Method filtering is case-insensitive
- POST/PUT/PATCH methods automatically display request bodies when method filter is applied
- Use `"/"` as pattern to match all requests

## What This Skill Does

Extracts HTTP/HTTPS request and response data from Charles Proxy session files, allowing you to:
- Filter requests by URL pattern (substring matching)
- Filter requests by HTTP method (GET, POST, PUT, PATCH, DELETE)
- View request bodies for mutation operations (POST/PUT/PATCH)
- Export extracted data to JSON files
- Generate traffic summaries with statistics
- Pretty-print JSON response bodies

## Input Requirements

**Required:**
- Path to Charles Proxy session file (.chlsj format)
- URL pattern to match (use "/" to match all requests)

**Optional:**
- HTTP method filter (GET, POST, PUT, PATCH, DELETE)
- Output mode (full, first-only, summary-only)
- Output file path for JSON export
- Pretty-print toggle for JSON formatting

## Output Format

**Summary mode:**
- Pattern match statistics
- Grouped paths with request counts
- Method and status code distribution

**Full mode:**
- Request details (method, path, status, timestamp)
- Request body (for POST/PUT/PATCH when method filter applied)
- Response body (JSON parsed or raw text)
- Pretty-printed JSON by default

**Export mode:**
- JSON file with structure:
  ```json
  {
    "pattern": "/api/endpoint",
    "total_requests": 10,
    "extracted_at": "ISO8601 timestamp",
    "requests": [...]
  }
  ```

## Common Usage Scenarios

**"Extract all /today responses from session.chlsj"**
→ Shows all requests matching /today pattern

**"Show POST requests to /logs with request bodies"**
→ Filters by POST method and displays request bodies

**"Export all /items responses to items.json"**
→ Saves filtered responses to JSON file

**"Summarize requests in the Charles session"**
→ Shows statistics without response bodies

## Limitations

- Only supports Charles Proxy JSON session format (.chlsj)
- Pattern matching is case-sensitive substring matching
- Method filtering is case-insensitive
- Large response bodies may be truncated in display (not in exports)
- Requires Python 3.x with standard library only (no external dependencies)

## Error Handling

The skill handles:
- Missing or inaccessible files (clear error message)
- Invalid JSON in session files (decoding error details)
- Empty result sets (informative message)
- Malformed request/response structures (graceful degradation)

## Troubleshooting

**"File not found" error:**
- Verify the .chlsj file path is correct
- Use absolute paths or ensure the file is in the current directory

**"Invalid JSON" error:**
- Ensure the file is a valid Charles Proxy session export
- Re-export the session from Charles Proxy

**No results found:**
- Pattern matching is case-sensitive - check capitalization
- Try using "/" to match all requests first
- Verify the endpoint exists in the session file using --summary-only

**Python not found:**
- Ensure Python 3.x is installed and available in PATH
- Try using `python` instead of `python3` or vice versa
