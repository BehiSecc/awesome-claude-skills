---
name: outline
description: "Search, read, and manage Outline wiki documents. Use when: (1) searching wiki for documentation, (2) reading wiki pages or articles, (3) listing wiki collections or documents, (4) creating or updating wiki content, (5) exporting documents as markdown. Works with any Outline wiki instance (self-hosted or cloud)."
license: Apache-2.0
metadata:
  author: sanjay3290
  version: "1.0"
---

# Outline Wiki Skill

Search, read, create, and manage documents in any Outline wiki instance. Works with all AI clients supporting the Agent Skills Standard.

## Requirements

- Python 3.8+
- Dependencies: `pip install -r requirements.txt`

## Setup

1. Get your API key from your Outline wiki:
   - Go to **Settings > API Tokens**
   - Create a new token with appropriate permissions

2. Configure the environment:
   ```bash
   cp .env.example .env
   # Edit .env with your API key
   ```

3. Set the environment variables:
   ```bash
   export OUTLINE_API_KEY=your-api-key-here
   # Optional: for self-hosted instances
   export OUTLINE_API_URL=https://your-wiki.example.com/api
   ```

## Usage

### Search documents
```bash
python3 scripts/outline.py search "deployment guide"
python3 scripts/outline.py search "API documentation" --limit 10
python3 scripts/outline.py search "onboarding" --collection-id <id>
```

### Read a document
```bash
python3 scripts/outline.py read <document-id>
python3 scripts/outline.py read <document-id> --json
```

### List collections
```bash
python3 scripts/outline.py list-collections
python3 scripts/outline.py list-collections --limit 50
```

### List documents in a collection
```bash
python3 scripts/outline.py list-documents --collection-id <id>
```

### Get collection details
```bash
python3 scripts/outline.py get-collection <collection-id>
```

### Create a document
```bash
python3 scripts/outline.py create --title "New Guide" --collection-id <id>
python3 scripts/outline.py create --title "Guide" --collection-id <id> --text "# Content here"
python3 scripts/outline.py create --title "Draft" --collection-id <id> --draft
```

### Update a document
```bash
python3 scripts/outline.py update <document-id> --title "Updated Title"
python3 scripts/outline.py update <document-id> --text "New content"
python3 scripts/outline.py update <document-id> --publish
```

### Export document as markdown
```bash
python3 scripts/outline.py export <document-id>
python3 scripts/outline.py export <document-id> --output doc.md
```

### Test authentication
```bash
python3 scripts/outline.py auth-info
```

## JSON Output

Add `--json` flag to any command for machine-readable output:
```bash
python3 scripts/outline.py search "query" --json
python3 scripts/outline.py read <id> --json
```

## Operations Reference

| Command | Description | Required Args |
|---------|-------------|---------------|
| search | Full-text search | query |
| read | Get document content | document_id |
| list-collections | List all collections | - |
| list-documents | List docs (optionally in collection) | - |
| get-collection | Get collection details | collection_id |
| create | Create new document | --title, --collection-id |
| update | Update existing document | document_id |
| export | Export as markdown | document_id |
| auth-info | Test API connection | - |

## Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| OUTLINE_API_KEY | Yes | - | Your Outline API token |
| OUTLINE_API_URL | No | https://app.getoutline.com/api | API URL |
| OUTLINE_TIMEOUT | No | 30 | Request timeout (seconds) |

## Troubleshooting

| Error | Solution |
|-------|----------|
| API key not configured | Set OUTLINE_API_KEY environment variable |
| Authentication failed | Verify API key is valid and not expired |
| Connection timeout | Check OUTLINE_API_URL and network connectivity |
| Document not found | Verify document ID is correct |
| Permission denied | Check API token has required permissions |

## Exit Codes

- **0**: Success
- **1**: Error (auth failed, not found, invalid request)

## Workflow

1. Run `auth-info` to verify connection
2. Run `list-collections` to see available collections
3. Run `search` or `list-documents` to find content
4. Run `read` to get full document content
5. Use `create`/`update` to modify wiki content
