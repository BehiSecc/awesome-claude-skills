---
name: azure-devops
description: |
  Manage Azure DevOps projects, work items, repos, PRs, pipelines, wikis, test plans, security alerts,
  variable groups, environments/approvals, branch policies, and attachments. Use when user asks to: manage
  sprints, create/update work items, list repos, create PRs, run pipelines, search code, manage wiki pages,
  check security alerts, manage variable groups, approve deployments, or configure branch policies.
  Covers 13 domains with 99 tools via REST API.
license: Apache-2.0
metadata:
  author: sanjay3290
  version: "1.0"
---

# Azure DevOps

Full Azure DevOps integration using OAuth or PAT authentication and REST API v7.1.

## First-Time Setup

### Option 1: OAuth (Recommended)

Login with OAuth device code flow (shows "Visual Studio Code" prompt):
```bash
python scripts/auth.py login --org MyOrganization
```
Follow the URL and enter the device code to authorize. Tokens auto-refresh.

### Option 2: PAT

Login with a Personal Access Token:
```bash
python scripts/auth.py login --org MyOrganization --pat YOUR_PAT
```

Create a PAT at `https://dev.azure.com/{org}/_usersSettings/tokens` with these scopes:
- **Work Items**: Read & Write
- **Code**: Read & Write (for repos/PRs)
- **Build**: Read & Execute (for pipelines)
- **Wiki**: Read & Write
- **Test Management**: Read & Write
- **Advanced Security**: Read (for security alerts)
- **Project and Team**: Read
- **Identity**: Read (for user search)

### Common Commands

Check authentication status:
```bash
python scripts/auth.py status
```

Logout:
```bash
python scripts/auth.py logout
```

## Core (scripts/core.py)

```bash
# List all projects
python scripts/core.py list-projects
python scripts/core.py list-projects --top 10

# List teams in a project
python scripts/core.py list-teams --project MyProject

# Search for a user identity
python scripts/core.py get-identity --search "john@example.com"
```

## Work Items (scripts/work_items.py)

```bash
# Get a work item
python scripts/work_items.py get --project MyProject --id 123
python scripts/work_items.py get --project MyProject --id 123 --expand relations

# Create a work item
python scripts/work_items.py create --project MyProject --type "User Story" --title "New feature"
python scripts/work_items.py create --project MyProject --type Bug --title "Fix login" \
  --field System.Description "Login fails on timeout" \
  --field System.AssignedTo "user@example.com"

# Update a work item
python scripts/work_items.py update --project MyProject --id 123 \
  --field System.State "Active" \
  --field System.AssignedTo "user@example.com"

# Batch get multiple work items
python scripts/work_items.py batch-get --project MyProject --ids 1,2,3

# Add children to a parent
python scripts/work_items.py add-children --project MyProject --parent-id 100 --child-ids 101,102

# Link work items
python scripts/work_items.py link --project MyProject --source-id 100 --target-id 200
python scripts/work_items.py link --project MyProject --source-id 100 --target-id 200 \
  --link-type "System.LinkTypes.Dependency-Forward"

# Remove a link
python scripts/work_items.py unlink --project MyProject --source-id 100 --relation-index 0

# Comments
python scripts/work_items.py add-comment --project MyProject --id 123 --text "Working on this"
python scripts/work_items.py list-comments --project MyProject --id 123

# History
python scripts/work_items.py get-revisions --project MyProject --id 123

# List work item types
python scripts/work_items.py list-types --project MyProject

# My assigned items
python scripts/work_items.py my-items --project MyProject

# Items in an iteration
python scripts/work_items.py iteration-items --project MyProject --iteration-path "MyProject\\Sprint 1"

# Backlogs
python scripts/work_items.py list-backlogs --project MyProject --team "MyProject Team"

# Saved queries
python scripts/work_items.py list-queries --project MyProject
python scripts/work_items.py get-query --project MyProject --path "Shared Queries/Active Bugs"
python scripts/work_items.py run-query --project MyProject --query-id "guid-here"

# WIQL query
python scripts/work_items.py run-wiql --project MyProject \
  --query "SELECT [System.Id], [System.Title] FROM WorkItems WHERE [System.State] = 'Active'"

# Delete / recycle bin
python scripts/work_items.py delete --project MyProject --id 999
python scripts/work_items.py recycle-bin --project MyProject
```

## Git Repos & PRs (scripts/repos.py)

```bash
# List repositories
python scripts/repos.py list --project MyProject

# Get repo details
python scripts/repos.py get --project MyProject --repo my-repo

# Branches
python scripts/repos.py list-branches --project MyProject --repo my-repo
python scripts/repos.py create-branch --project MyProject --repo my-repo --name feature/new --source main

# Commits
python scripts/repos.py search-commits --project MyProject --repo my-repo --path /src --author "john" --top 10

# Pull requests
python scripts/repos.py list-prs --project MyProject --repo my-repo
python scripts/repos.py list-prs --project MyProject --status completed --top 5

python scripts/repos.py create-pr --project MyProject --repo my-repo \
  --source feature/new --target main --title "Add new feature" --description "Details here"

python scripts/repos.py get-pr --project MyProject --repo my-repo --pr-id 42

python scripts/repos.py update-pr --project MyProject --repo my-repo --pr-id 42 --title "Updated title"

# Reviewers
python scripts/repos.py list-reviewers --project MyProject --repo my-repo --pr-id 42
python scripts/repos.py add-reviewer --project MyProject --repo my-repo --pr-id 42 \
  --reviewer-id "guid" --vote 10

# PR comments
python scripts/repos.py list-threads --project MyProject --repo my-repo --pr-id 42
python scripts/repos.py create-thread --project MyProject --repo my-repo --pr-id 42 \
  --content "Looks good!" --file-path "/src/main.py" --line 25
python scripts/repos.py add-thread-comment --project MyProject --repo my-repo --pr-id 42 \
  --thread-id 1 --content "Fixed"

# Complete or abandon PR
python scripts/repos.py complete-pr --project MyProject --repo my-repo --pr-id 42
python scripts/repos.py complete-pr --project MyProject --repo my-repo --pr-id 42 \
  --merge-strategy rebase --keep-source
python scripts/repos.py abandon-pr --project MyProject --repo my-repo --pr-id 42

# Diff
python scripts/repos.py get-diff --project MyProject --repo my-repo --base main --target feature/new

# Browse files
python scripts/repos.py list-files --project MyProject --repo my-repo --path /src --branch main
```

## Iterations & Capacity (scripts/work.py)

```bash
# List iterations
python scripts/work.py list-iterations --project MyProject

# Create iteration
python scripts/work.py create-iteration --project MyProject --name "Sprint 5" \
  --start-date 2026-03-01 --finish-date 2026-03-14

# Get iteration details
python scripts/work.py get-iteration --project MyProject --iteration-id "guid"

# Team iterations
python scripts/work.py team-iterations --project MyProject --team "MyTeam" --timeframe current

# Assign iteration to team
python scripts/work.py assign-iteration --project MyProject --team "MyTeam" --iteration-id "guid"

# Capacities
python scripts/work.py get-capacities --project MyProject --team "MyTeam" --iteration-id "guid"
python scripts/work.py set-capacity --project MyProject --team "MyTeam" --iteration-id "guid" \
  --member-id "user-guid" --activity Development --capacity-per-day 6
```

## Pipelines & Builds (scripts/pipelines.py)

```bash
# Create pipeline
python scripts/pipelines.py create --project MyProject --name "CI" \
  --repo-id "repo-guid" --yaml-path "/azure-pipelines.yml"

# List builds
python scripts/pipelines.py list-builds --project MyProject --top 5
python scripts/pipelines.py list-builds --project MyProject --status completed --branch main

# Get build details and logs
python scripts/pipelines.py get-build --project MyProject --build-id 100
python scripts/pipelines.py build-logs --project MyProject --build-id 100
python scripts/pipelines.py build-logs --project MyProject --build-id 100 --log-id 3
python scripts/pipelines.py build-changes --project MyProject --build-id 100

# Pipeline definitions
python scripts/pipelines.py list-definitions --project MyProject
python scripts/pipelines.py list-definitions --project MyProject --name "CI"

# Run a pipeline
python scripts/pipelines.py run --project MyProject --pipeline-id 5 --branch develop
python scripts/pipelines.py run --project MyProject --pipeline-id 5 \
  --variable ENV production --variable DEPLOY true

# Pipeline runs
python scripts/pipelines.py get-run --project MyProject --pipeline-id 5 --run-id 200
python scripts/pipelines.py list-runs --project MyProject --pipeline-id 5 --top 10

# Stage management
python scripts/pipelines.py update-stage --project MyProject --build-id 100 --stage Deploy --state retry

# Artifacts
python scripts/pipelines.py get-artifacts --project MyProject --build-id 100

# Queue and cancel builds
python scripts/pipelines.py queue-build --project MyProject --definition-id 10 --branch main
python scripts/pipelines.py cancel-build --project MyProject --build-id 100

# Latest build status
python scripts/pipelines.py build-status --project MyProject --definition-id 10
```

## Search (scripts/search.py)

```bash
# Search code
python scripts/search.py code --project MyProject --query "TODO"
python scripts/search.py code --project MyProject --query "connectionString" --top 10

# Search wiki
python scripts/search.py wiki --project MyProject --query "deployment guide"

# Search work items
python scripts/search.py work-items --project MyProject --query "login bug"
```

## Wiki (scripts/wiki.py)

```bash
# List wikis
python scripts/wiki.py list --project MyProject

# Get wiki details
python scripts/wiki.py get --project MyProject --wiki-id MyProject.wiki

# List pages
python scripts/wiki.py list-pages --project MyProject --wiki-id MyProject.wiki
python scripts/wiki.py list-pages --project MyProject --wiki-id MyProject.wiki --path /Architecture

# Get page content
python scripts/wiki.py get-page --project MyProject --wiki-id MyProject.wiki --path /Home
python scripts/wiki.py get-page-content --project MyProject --wiki-id MyProject.wiki --path /Home

# Create or update a page
python scripts/wiki.py create-or-update-page --project MyProject --wiki-id MyProject.wiki \
  --path /NewPage --content "# New Page\n\nContent here"
```

## Test Plans (scripts/test_plans.py)

```bash
# Test plans
python scripts/test_plans.py list-plans --project MyProject
python scripts/test_plans.py get-plan --project MyProject --plan-id 1
python scripts/test_plans.py create-plan --project MyProject --name "Release 2.0 Tests"

# Test suites
python scripts/test_plans.py list-suites --project MyProject --plan-id 1
python scripts/test_plans.py get-suite --project MyProject --plan-id 1 --suite-id 2

# Test cases
python scripts/test_plans.py list-test-cases --project MyProject --plan-id 1 --suite-id 2
python scripts/test_plans.py add-test-cases --project MyProject --plan-id 1 --suite-id 2 --ids 100,101

# Test results
python scripts/test_plans.py list-results --project MyProject --run-id 50
python scripts/test_plans.py get-result --project MyProject --run-id 50 --result-id 1
```

## Advanced Security (scripts/security.py)

```bash
# List security alerts
python scripts/security.py list-alerts --project MyProject --repo my-repo
python scripts/security.py list-alerts --project MyProject --repo my-repo \
  --state active --severity high --type dependency

# Get alert details
python scripts/security.py get-alert --project MyProject --repo my-repo --alert-id 42
```

## Variable Groups (scripts/variable_groups.py)

```bash
# List variable groups
python scripts/variable_groups.py list --project MyProject

# Get a variable group
python scripts/variable_groups.py get --project MyProject --id 1

# Create a variable group
python scripts/variable_groups.py create --project MyProject --name "Deploy Vars" \
  --variable ENV production --variable REGION eastus --description "Deploy config"

# Update a variable group
python scripts/variable_groups.py update --project MyProject --id 1 --name "New Name" \
  --variable ENV staging

# Add/update a single variable (supports --secret for sensitive values)
python scripts/variable_groups.py add-variable --project MyProject --id 1 \
  --key API_KEY --value "secret123" --secret

# Remove a variable
python scripts/variable_groups.py remove-variable --project MyProject --id 1 --key OLD_VAR

# Delete a variable group
python scripts/variable_groups.py delete --project MyProject --id 1
```

## Environments & Approvals (scripts/environments.py)

```bash
# List environments
python scripts/environments.py list --project MyProject

# Create an environment
python scripts/environments.py create --project MyProject --name "Production" \
  --description "Production environment"

# Get environment details
python scripts/environments.py get --project MyProject --id 1

# List checks/approvals configured on an environment
python scripts/environments.py list-checks --project MyProject --id 1

# List pending approvals
python scripts/environments.py list-approvals --project MyProject
python scripts/environments.py list-approvals --project MyProject --state all

# Approve/reject
python scripts/environments.py approve --project MyProject --approval-id "guid" \
  --comment "Looks good"
python scripts/environments.py reject --project MyProject --approval-id "guid" \
  --comment "Needs fixes"

# Delete an environment
python scripts/environments.py delete --project MyProject --id 1
```

## Branch Policies (scripts/policies.py)

```bash
# List all policies
python scripts/policies.py list --project MyProject
python scripts/policies.py list --project MyProject --branch main --repo my-repo-id

# Get a policy
python scripts/policies.py get --project MyProject --id 1

# List available policy types
python scripts/policies.py list-types --project MyProject

# Create minimum reviewers policy
python scripts/policies.py create-min-reviewers --project MyProject \
  --repo-id "repo-guid" --branch main --min-reviewers 2 --reset-on-push

# Create build validation policy
python scripts/policies.py create-build-policy --project MyProject \
  --repo-id "repo-guid" --branch main --build-definition-id 10

# Update a policy (enable/disable/blocking)
python scripts/policies.py update --project MyProject --id 1 --enabled false
python scripts/policies.py update --project MyProject --id 1 --blocking true

# List policy evaluations for a PR
python scripts/policies.py list-evaluations --project MyProject --pr-id 42

# Delete a policy
python scripts/policies.py delete --project MyProject --id 1
```

## Work Item Attachments (scripts/attachments.py)

```bash
# Upload a file
python scripts/attachments.py upload --project MyProject --file ./report.pdf

# Attach an uploaded file to a work item
python scripts/attachments.py attach --project MyProject --id 123 \
  --url "https://dev.azure.com/org/project/_apis/wit/attachments/guid" \
  --comment "Test report"

# Upload and attach in one step
python scripts/attachments.py upload-and-attach --project MyProject --id 123 \
  --file ./screenshot.png --comment "Bug screenshot"

# List attachments on a work item
python scripts/attachments.py list --project MyProject --id 123

# Download an attachment
python scripts/attachments.py download --project MyProject \
  --url "https://dev.azure.com/org/project/_apis/wit/attachments/guid" \
  --output ./downloaded.pdf

# Remove an attachment by relation index
python scripts/attachments.py remove --project MyProject --id 123 --index 0
```

## Common Work Item Fields

| Field | Reference Name |
|-------|---------------|
| Title | `System.Title` |
| State | `System.State` |
| Assigned To | `System.AssignedTo` |
| Description | `System.Description` |
| Area Path | `System.AreaPath` |
| Iteration Path | `System.IterationPath` |
| Priority | `Microsoft.VSTS.Common.Priority` |
| Story Points | `Microsoft.VSTS.Scheduling.StoryPoints` |
| Tags | `System.Tags` |
| Repro Steps | `Microsoft.VSTS.TCM.ReproSteps` |
| Acceptance Criteria | `Microsoft.VSTS.Common.AcceptanceCriteria` |

## Link Types

| Link Type | Reference Name |
|-----------|---------------|
| Related | `System.LinkTypes.Related` |
| Parent → Child | `System.LinkTypes.Hierarchy-Forward` |
| Child → Parent | `System.LinkTypes.Hierarchy-Reverse` |
| Predecessor | `System.LinkTypes.Dependency-Forward` |
| Successor | `System.LinkTypes.Dependency-Reverse` |

## Token Management

Credentials stored securely using the system keyring:
- **macOS**: Keychain
- **Windows**: Windows Credential Locker
- **Linux**: Secret Service API

Service name: `azure-devops-skill`
