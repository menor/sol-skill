---
name: sol
description: Use when managing Upsun projects, environments, variables, deployments, backups, resources, or SSH access. Helps with listing projects, creating branches, setting variables, viewing activities, managing backups, scaling resources, and redeploying.
allowed-tools: Bash, Read
---

# Sol - Upsun CLI for AI Agents

Sol is an agent-optimized CLI for Upsun. Use it to manage Upsun projects, environments, variables, and deployments.

## Output Optimization

Sol minimizes token usage for agents:

- **Lean output by default**: List commands return essential fields only (up to 99% smaller)
- **TOON format**: Token-efficient encoding (~50% smaller than JSON)
- **`--full` flag**: Get all fields when needed

| Command | Default fields | Size reduction |
|---------|---------------|----------------|
| `project:list` | id, title, region | 82% smaller |
| `environment:list` | id, name, status, parent | 99% smaller |
| `activity:list` | id, type, state, created_at | 90% smaller |

Use `-o json` if humans need to read the output. Use `--full` when you need all fields.

## Prerequisites

The `sol` binary must be installed and available in PATH. The user must be authenticated (`sol auth:login`).

## Discovering Commands

Use `--schema` to discover available commands and their usage:

```bash
# List all available commands
sol --schema

# Get detailed schema for a specific command
sol project:list --schema
sol variable:set --schema
```

The schema includes flags, arguments, output format, examples, and exit codes.

## Common Workflows

### List and Inspect Projects

```bash
# List all projects (returns: id, title, region)
sol project:list

# List projects with all fields
sol project:list --full

# Get project details
sol project:info PROJECT_ID
```

### Manage Environments

```bash
# List environments (returns: id, name, status, parent)
sol environment:list -p PROJECT_ID

# List environments with all fields
sol environment:list -p PROJECT_ID --full

# Get environment details
sol environment:info ENV_NAME -p PROJECT_ID

# Create a new branch environment
sol environment:branch NEW_ENV -p PROJECT_ID --parent main

# Activate/deactivate environments
sol environment:activate ENV_NAME -p PROJECT_ID
sol environment:deactivate ENV_NAME -p PROJECT_ID
```

### Manage Variables

```bash
# List variables (project or environment level)
sol variable:list -p PROJECT_ID --level project
sol variable:list -p PROJECT_ID -e ENV_NAME --level environment

# Get a variable value
sol variable:get VAR_NAME -p PROJECT_ID --level project

# Set a variable
sol variable:set VAR_NAME "value" -p PROJECT_ID --level project

# Delete a variable
sol variable:delete VAR_NAME -p PROJECT_ID --level project
```

### View Activities

```bash
# List recent activities (returns: id, type, state, created_at)
sol activity:list -p PROJECT_ID --limit 10

# List activities with all fields (result, description, timestamps, etc.)
sol activity:list -p PROJECT_ID --limit 10 --full

# Filter by state (pending, in_progress, complete)
sol activity:list -p PROJECT_ID --state in_progress

# Filter by result (success, failure)
sol activity:list -p PROJECT_ID --result failure --limit 10

# View activity log
sol activity:log ACTIVITY_ID -p PROJECT_ID
```

### Deployments

```bash
# Redeploy an environment
sol redeploy -p PROJECT_ID -e ENV_NAME

# Push code (triggers deployment)
sol push -p PROJECT_ID -e ENV_NAME
```

### SSH Access

```bash
# SSH into an environment
sol ssh -p PROJECT_ID -e ENV_NAME

# SSH to a specific app
sol ssh -p PROJECT_ID -e ENV_NAME --app myapp
```

## Multi-Step Workflows

These scenarios require multiple commands. Claude should orchestrate them based on user intent.

### Debug a Failed Deployment

When a user says "my deployment failed" or "why isn't my site working":

1. **Find the project** (if not specified):
   ```bash
   sol project:list
   ```

2. **Check recent activities for failures**:
   ```bash
   sol activity:list -p PROJECT_ID --result failure --limit 5
   ```

3. **Get the activity log to identify the error**:
   ```bash
   sol activity:log ACTIVITY_ID -p PROJECT_ID
   ```

4. **Report findings**: Look for build errors, deployment failures, or resource issues in the log output.

### Set Up a Feature Branch Environment

When a user says "create a new environment for feature X" or "I need a staging copy":

1. **List existing environments to find the parent**:
   ```bash
   sol environment:list -p PROJECT_ID
   ```

2. **Create the branch environment**:
   ```bash
   sol environment:branch feature-x -p PROJECT_ID --parent main --title "Feature X Development"
   ```

3. **Copy any environment-specific variables if needed**:
   ```bash
   # Get variables from parent
   sol variable:list -p PROJECT_ID -e main --level environment

   # Set on new environment (for each variable that needs copying)
   sol variable:set VAR_NAME "value" -p PROJECT_ID -e feature-x --level environment
   ```

4. **Verify the environment is active**:
   ```bash
   sol environment:info feature-x -p PROJECT_ID
   ```

### Compare Variables Between Environments

When a user says "what's different between staging and production" or "compare env vars":

1. **Get variables from first environment**:
   ```bash
   sol variable:list -p PROJECT_ID -e staging --level environment
   ```

2. **Get variables from second environment**:
   ```bash
   sol variable:list -p PROJECT_ID -e main --level environment
   ```

3. **Compare and report**: Identify variables that exist in one but not the other, or have different values.

### Find and Update a Variable Across Environments

When a user says "update DATABASE_URL everywhere" or "change API_KEY on all environments":

1. **List all environments**:
   ```bash
   sol environment:list -p PROJECT_ID
   ```

2. **Check which environments have the variable**:
   ```bash
   # For each environment
   sol variable:get VAR_NAME -p PROJECT_ID -e ENV_NAME --level environment
   ```

3. **Update on each environment** (confirm with user first):
   ```bash
   sol variable:set VAR_NAME "new-value" -p PROJECT_ID -e ENV_NAME --level environment
   ```

### Troubleshoot an Inactive Environment

When a user says "my environment isn't working" or "site is down":

1. **Check environment status**:
   ```bash
   sol environment:info ENV_NAME -p PROJECT_ID
   ```

2. **If status is "inactive", activate it**:
   ```bash
   sol environment:activate ENV_NAME -p PROJECT_ID
   ```

3. **If status is "active" but issues persist, check recent activities**:
   ```bash
   sol activity:list -p PROJECT_ID -e ENV_NAME --limit 5
   ```

4. **SSH in to investigate if needed**:
   ```bash
   sol ssh -p PROJECT_ID -e ENV_NAME
   ```

### Clean Up Old Feature Branches

When a user says "clean up old environments" or "delete merged branches":

1. **List all environments**:
   ```bash
   sol environment:list -p PROJECT_ID
   ```

2. **Identify inactive or old environments** (look at status and last activity).

3. **For each environment to remove** (confirm with user first):
   ```bash
   # Deactivate first if active
   sol environment:deactivate ENV_NAME -p PROJECT_ID

   # Then delete
   sol environment:delete ENV_NAME -p PROJECT_ID
   ```

### Redeploy After Configuration Change

When a user says "I updated the config, redeploy" or "apply my changes":

1. **Check current environment status**:
   ```bash
   sol environment:info ENV_NAME -p PROJECT_ID
   ```

2. **Trigger redeploy**:
   ```bash
   sol redeploy -p PROJECT_ID -e ENV_NAME
   ```

3. **Monitor the deployment**:
   ```bash
   sol activity:list -p PROJECT_ID -e ENV_NAME --state in_progress --limit 1
   ```

4. **Once complete, verify success**:
   ```bash
   sol activity:list -p PROJECT_ID -e ENV_NAME --state complete --limit 1
   ```

### Merge a Feature Branch

When a user says "merge my feature branch" or "merge staging into main":

1. **Check environment status**:
   ```bash
   sol environment:info feature-x -p PROJECT_ID
   ```

2. **Merge into parent**:
   ```bash
   sol environment:merge feature-x -p PROJECT_ID --wait
   ```

3. **Verify the merge completed**:
   ```bash
   sol activity:list -p PROJECT_ID --limit 1
   ```

### Sync from Parent Environment

When a user says "sync my branch with main" or "pull latest from production":

1. **Check what will be synced**:
   ```bash
   sol environment:info feature-x -p PROJECT_ID
   ```

2. **Sync code and/or data**:
   ```bash
   # Sync both code and data
   sol environment:sync feature-x -p PROJECT_ID --code --data

   # Or sync code only
   sol environment:sync feature-x -p PROJECT_ID --code
   ```

3. **Verify sync completed**:
   ```bash
   sol activity:list -p PROJECT_ID -e feature-x --limit 1
   ```

### Backup and Restore

When a user says "backup my environment" or "I need to restore from backup":

1. **Create a backup**:
   ```bash
   # Standard backup
   sol backup:create -p PROJECT_ID -e main

   # Safe backup (waits for running activities)
   sol backup:create -p PROJECT_ID -e main --safe
   ```

2. **List available backups**:
   ```bash
   sol backup:list -p PROJECT_ID -e main
   ```

3. **Restore from backup**:
   ```bash
   # Restore to same environment
   sol backup:restore BACKUP_ID -p PROJECT_ID -e main

   # Restore to a different environment
   sol backup:restore BACKUP_ID -p PROJECT_ID -e main --target staging
   ```

### Check Services and Resources

When a user says "what services are running" or "check my app resources":

1. **List services**:
   ```bash
   sol service:list -p PROJECT_ID -e main
   ```

2. **List applications**:
   ```bash
   sol app:list -p PROJECT_ID -e main
   ```

3. **Check resource allocation**:
   ```bash
   sol resources:get -p PROJECT_ID -e main
   ```

4. **Adjust resources if needed**:
   ```bash
   sol resources:set -p PROJECT_ID -e main --service db --cpu 1 --memory 2048
   ```

### Get Environment URLs and Routes

When a user says "what's the URL" or "show me the routes":

1. **Get primary URL**:
   ```bash
   sol environment:url -p PROJECT_ID -e main
   ```

2. **List all routes**:
   ```bash
   sol route:list -p PROJECT_ID -e main
   ```

3. **Check service relationships**:
   ```bash
   sol environment:relationships -p PROJECT_ID -e main
   ```

### Manage Integrations

When a user says "what integrations are set up" or "check GitHub integration":

1. **List all integrations**:
   ```bash
   sol integration:list -p PROJECT_ID
   ```

2. **Get integration details**:
   ```bash
   sol integration:get INTEGRATION_ID -p PROJECT_ID
   ```

### Check Domains and Certificates

When a user says "what domains are configured" or "check SSL certificates":

1. **List domains**:
   ```bash
   sol domain:list -p PROJECT_ID
   ```

2. **List certificates**:
   ```bash
   sol certificate:list -p PROJECT_ID
   ```

### Manage SSH Keys

When a user says "list my SSH keys" or "check SSH access":

1. **List SSH keys**:
   ```bash
   sol ssh-key:list
   ```

### Organization Overview

When a user says "what organizations do I have access to":

1. **List organizations**:
   ```bash
   sol organization:list
   ```

2. **Get organization details**:
   ```bash
   sol organization:info ORG_ID
   ```

3. **List users on a project**:
   ```bash
   sol user:list -p PROJECT_ID
   ```

## Global Flags

These flags work with all commands:

| Flag | Short | Description |
|------|-------|-------------|
| `--output` | `-o` | Output format: toon (default), json |
| `--project` | `-p` | Project ID (or set UPSUN_PROJECT env var) |
| `--environment` | `-e` | Environment name (or set UPSUN_ENVIRONMENT env var) |
| `--quiet` | `-q` | Suppress non-essential output |
| `--no-cache` | | Bypass response cache |
| `--debug` | | Show API request/response details |
| `--schema` | | Output command schema instead of running |

## Command-Specific Flags

| Flag | Short | Commands | Description |
|------|-------|----------|-------------|
| `--full` | `-f` | `project:list`, `environment:list`, `activity:list`, etc. | Include all fields in output |
| `--result` | | `activity:list` | Filter by result (success, failure) |
| `--wait` | `-w` | `environment:branch`, `environment:merge`, `environment:sync`, `redeploy`, etc. | Wait for activity to complete |
| `--safe` | | `backup:create` | Wait for running activities before backup |
| `--target` | | `backup:restore` | Restore to a different environment |
| `--code` | | `environment:sync` | Sync code from parent |
| `--data` | | `environment:sync` | Sync data from parent |
| `--app` | `-a` | `ssh`, `environment:relationships`, `resources:set` | Target specific application |
| `--service` | `-s` | `resources:set` | Target specific service |

## Error Handling

Sol uses structured errors with codes. Check the exit code and parse the error output:

```
[error]
code: NOT_FOUND
message: Project not found
hint: Check the project ID or run 'sol project:list' to see available projects
```

Common exit codes:
- 0: Success
- 1: General error
- 2: Authentication error
- 3: Not found
- 4: Validation error

## Best Practices

1. **Use --schema first**: When unsure about a command, check its schema
2. **Set project context**: Use `-p PROJECT_ID` or set `UPSUN_PROJECT` env var
3. **Check authentication**: Run `sol auth:info` to verify auth status
4. **Use -o json for humans**: When showing output to users who need to read it
5. **Use --quiet for scripts**: Suppress progress messages with `-q`
