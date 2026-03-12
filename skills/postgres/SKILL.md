---
name: postgres
description: "Execute read-only SQL queries against multiple PostgreSQL databases. Use when: (1) querying PostgreSQL databases, (2) exploring database schemas/tables, (3) running SELECT queries for data analysis, (4) checking database contents. Supports multiple database connections with descriptions for intelligent auto-selection. Blocks all write operations (INSERT, UPDATE, DELETE, DROP, etc.) for safety."
license: Apache-2.0
metadata:
  author: sanjay3290
  version: "1.0"
---

# PostgreSQL Read-Only Query Skill

Execute safe, read-only queries against configured PostgreSQL databases.

## Requirements

- Python 3.8+
- psycopg2-binary: `pip install -r requirements.txt`

## Setup

Create `connections.json` in the skill directory or `~/.config/claude/postgres-connections.json`.

**Security**: Set file permissions to `600` since it contains credentials:
```bash
chmod 600 connections.json
```

```json
{
  "databases": [
    {
      "name": "production",
      "description": "Main app database - users, orders, transactions",
      "host": "db.example.com",
      "port": 5432,
      "database": "app_prod",
      "user": "readonly_user",
      "password": "your-password",
      "sslmode": "require"
    }
  ]
}
```

### Config Fields

| Field | Required | Description |
|-------|----------|-------------|
| name | Yes | Identifier for the database (case-insensitive) |
| description | Yes | What data this database contains (used for auto-selection) |
| host | Yes | Database hostname |
| port | No | Port number (default: 5432) |
| database | Yes | Database name |
| user | Yes | Username |
| password | Yes | Password |
| sslmode | No | SSL mode: disable, allow, prefer (default), require, verify-ca, verify-full |

## Usage

### List configured databases
```bash
python3 scripts/query.py --list
```

### Query a database
```bash
python3 scripts/query.py --db production --query "SELECT * FROM users LIMIT 10"
```

### List tables
```bash
python3 scripts/query.py --db production --tables
```

### Show schema
```bash
python3 scripts/query.py --db production --schema
```

### Limit results
```bash
python3 scripts/query.py --db production --query "SELECT * FROM orders" --limit 100
```

## Database Selection

Match user intent to database `description`:

| User asks about | Look for description containing |
|-----------------|--------------------------------|
| users, accounts | users, accounts, customers |
| orders, sales | orders, transactions, sales |
| analytics, metrics | analytics, metrics, reports |
| logs, events | logs, events, audit |

If unclear, run `--list` and ask user which database.

## Safety Features

- **Read-only session**: Connection uses PostgreSQL `readonly=True` mode (primary protection)
- **Query validation**: Only SELECT, SHOW, EXPLAIN, WITH queries allowed
- **Single statement**: Multiple statements per query rejected
- **SSL support**: Configurable SSL mode for encrypted connections
- **Query timeout**: 30-second statement timeout enforced
- **Memory protection**: Max 10,000 rows per query to prevent OOM
- **Column width cap**: 100 char max per column for readable output
- **Credential sanitization**: Error messages don't leak passwords

## Troubleshooting

| Error | Solution |
|-------|----------|
| Config not found | Create `connections.json` in skill directory |
| Authentication failed | Check username/password in config |
| Connection timeout | Verify host/port, check firewall/VPN |
| SSL error | Try `"sslmode": "disable"` for local databases |
| Permission warning | Run `chmod 600 connections.json` |

## Exit Codes

- **0**: Success
- **1**: Error (config missing, auth failed, invalid query, database error)

## Workflow

1. Run `--list` to show available databases
2. Match user intent to database description
3. Run `--tables` or `--schema` to explore structure
4. Execute query with appropriate LIMIT
