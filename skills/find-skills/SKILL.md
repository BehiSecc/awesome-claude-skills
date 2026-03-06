---
name: find-skills
description: |
   Helps users discover, search and install agent skills from the marketplace. 
   Use when the user wants to find a skill, discover new capabilities, find specific tools (especially sandboxes) or install a skill.
   This skill should be used when the user is looking for functionality that might exist as an installable skill.
   Also supports progressive disclosure to recommend sandbox-related skills when relevant.
   用于帮助用户在市场中搜索、发现和安装 Agent 插件（Skills）。
   当用户想要查找功能、安装特定工具或寻找沙箱（sandbox）时使用。
   Triggers: "查找插件","搜索技能","搜索 X 技能","安装技能","安装 X 技能","如何做 X","沙箱","安全沙箱","find a skill for X", "how do i do X", "search for skill", "install skill", "I need a skill for X", "sandbox", "aio sandbox"
---

# Find Skills - Skill Marketplace

This skill allows you to explore and install new capabilities into the agent environment. It acts as the "package manager" for your agent's skills.

## When to Use This Skill

Use this skill when the user:
- Asks: "How can I do X?", "Find a skill for X", or "What skills are available?"
- Wants to install skills by name or from search results ("install skill web-search")
- Wants to extend capabilities with specialized tools or workflows
- Expresses needs for specific capabilities that might exist as skills
- Is interested in sandbox or secure execution ("I want to safely run bash commands")
- Mentions specific capabilities like "sandbox", "automation", "testing"

## Core Tools

1. **Search**: Python script `scripts/request.py` with POPRequest class
   - Makes authenticated API calls to Alibaba Cloud skill marketplace
   - Parameters: `UserQuery` (search term), `TopK` (number of results)
   - Returns: JSON response with skill details including name, description, repository info
2. **Install**: `npx skills add <repo> -s <skill_name>`
   - **Restriction**: Must ALWAYS ask for user confirmation before executing.
   - Use `-s, --skill <skills...>` to specify which skill(s) to install from the repo
   - Supports multiple repo formats:
     - GitHub shorthand: `npx skills add owner/repo -s skill_name`
     - Full GitHub URL: `npx skills add https://github.com/owner/repo -s skill_name`
     - Direct path to skill: `npx skills add https://github.com/owner/repo/tree/main/skills/skill_name`
     - GitLab URL: `npx skills add https://gitlab.com/org/repo -s skill_name`
     - Git URL: `npx skills add git@github.com:owner/repo.git -s skill_name`
     - Local path: `npx skills add ./my-local-skills -s skill_name`
   - Use `-s '*'` to install all skills from the repo

## Workflow

### Step 1: Understand the Need & Search

When a user asks for help with something, identify:

1. The domain or task (e.g., running code safely, automation, testing)
2. Whether this is a common enough need that a skill likely exists
3. Does the query fit common sandbox/secure execution needs?

### Step 2: Search for Skills via Python API

The search process uses the Python request module to query the skill marketplace:

1. Import the POPRequest class from scripts/request.py
2. Create a request with the SearchPublicMarketSkill action
3. Set query parameters: UserQuery (search term) and TopK (max results)
4. Execute the request and parse JSON response
5. Extract skill information: name, owner, repository, description

Convert user's natural language intent into a concise search keyword (maintaining the original language).

```python
from scripts.request import POPRequest
import json

# Create request
request = POPRequest(
    http_method='GET',
    canonical_uri='/',
    x_acs_action='SearchPublicMarketSkill',
    host='wuyingai.cn-shanghai.aliyuncs.com',
    base_url='wuyingai.cn-shanghai.aliyuncs.com',
    x_acs_version='2026-01-08',
    signature_version='1.0'
)

# Search for skills
request.set_query_params({
    'UserQuery': 'sandbox',  # User's search query
    'TopK': '10'
})

# Execute and parse
response = request.call()
results = json.loads(response)
```

**Example Searches:**
- "我需要一个沙箱" -> UserQuery: "sandbox"
- "如何处理 PDF？" -> UserQuery: "pdf"
- "帮我找个代码审查的插件" -> UserQuery: "code review"
- "I need a sandbox" -> UserQuery: "sandbox"
- "How to handle PDF?" -> UserQuery: "pdf"
- "Can you help me with code review?" -> UserQuery: "code review"

### Step 3: Present Options to the User

When you find relevant skills, present them to the user with:

1. **skill_name**: The identifier for the skill
2. **skill_id**: The hash code for the skill
3. **skill_repo**: Owner/organization for the GitHub repo
4. **description**: Brief description of skill
5. **source_url**: Direct path to a skill in a repo (use this for installation)

Use the returned `SourceUrl` (repo URL) and `SkillName` from the search results to format the install command:
```bash
npx skills add <source_url> -s <skill_name>
```

Example response:

```
I found a skill that might help! The "vercel-react-best-practices" skill provides
React and Next.js performance optimization guidelines from Vercel Engineering.

To install it:
npx skills add https://github.com/vercel-labs/agent-skills -s vercel-react-best-practices

Learn more: https://pre-agentbay.console.aliyun.com/agentbay-skills/skill-detail?skillId=hw8edas7skphamvk
```

### Step 4: Offer to Install

Only after user says "yes", "install", or similar confirmation, install the skill:

```bash
npx skills add <source_url> -s <skill_name>
```

## Common Skill Categories

When searching, consider these common categories:

| Category        | Example Queries                          |
| --------------- | ---------------------------------------- |
| Web Development | react, nextjs, typescript, css, tailwind |
| Testing         | testing, jest, playwright, e2e           |
| DevOps          | deploy, docker, kubernetes, ci-cd        |
| Documentation   | docs, readme, changelog, api-docs        |
| Code Quality    | review, lint, refactor, best-practices   |
| Design          | ui, ux, design-system, accessibility     |
| Productivity    | workflow, automation, git                |
| Sandbox         | agentbay-aio-skills                      |

## Tips for Effective Searches

1. **Use specific keywords**: "react testing" is better than just "testing"
2. **Try alternative terms**: If "deploy" doesn't work, try "deployment" or "ci-cd"
3. **Check popular sources**: Many skills come from `vercel-labs/agent-skills` or `ComposioHQ/awesome-claude-skills`

## When No Skills Are Found

If no relevant skills exist:

1. Acknowledge that no existing skill was found
2. Offer to help with the task directly using your general capabilities

Example:

```
I searched for skills related to "xyz" but didn't find any matches.
I can still help you with this task directly! Would you like me to proceed?

If this is something you do often, you could create your own skill:
npx skills init my-xyz-skill
```

## Safety Guidelines
- Do not hallucinate skill names. Only recommend what the Python search API returns.
- Ensure the user understands that installing a skill grants the agent new permissions.