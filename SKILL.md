---
name: sol
description: Use when managing Upsun projects, environments, variables, deployments, or SSH access. Helps with listing projects, creating branches, setting variables, viewing activities, and redeploying.
allowed-tools: Bash, Read
---

# Sol - Upsun CLI for AI Agents

Sol is an agent-optimized CLI for Upsun. Use it to manage Upsun projects, environments, variables, and deployments.

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

## Output Formats

Sol supports two output formats:

- **JSON** (default): Standard JSON, best for parsing
- **TOON**: Token-Oriented Object Notation, ~50% fewer tokens

Use `--output toon` when you need to conserve tokens:

```bash
sol project:list --output toon
sol environment:list -p PROJECT_ID --output toon
```

## Common Workflows

### List and Inspect Projects

```bash
# List all projects
sol project:list

# Get project details
sol project:info PROJECT_ID
```

### Manage Environments

```bash
# List environments for a project
sol environment:list -p PROJECT_ID

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
# List recent activities
sol activity:list -p PROJECT_ID --limit 10

# Filter by state
sol activity:list -p PROJECT_ID --state in_progress

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

## Global Flags

These flags work with all commands:

| Flag | Short | Description |
|------|-------|-------------|
| `--project` | `-p` | Project ID (or set UPSUN_PROJECT env var) |
| `--environment` | `-e` | Environment name (or set UPSUN_ENVIRONMENT env var) |
| `--output` | `-o` | Output format: json, toon |
| `--quiet` | `-q` | Suppress non-essential output |
| `--no-cache` | | Bypass response cache |
| `--debug` | | Show API request/response details |
| `--schema` | | Output command schema instead of running |

## Error Handling

Sol uses structured errors with codes. Check the exit code and parse the JSON error output:

```json
{
  "error": {
    "code": "NOT_FOUND",
    "message": "Project not found",
    "hint": "Check the project ID or run 'sol project:list' to see available projects"
  }
}
```

Common exit codes:
- 0: Success
- 1: General error
- 2: Authentication error
- 3: Not found
- 4: Validation error

## Best Practices

1. **Use --schema first**: When unsure about a command, check its schema
2. **Prefer TOON output**: Use `--output toon` to reduce token usage
3. **Set project context**: Use `-p PROJECT_ID` or set `UPSUN_PROJECT` env var
4. **Check authentication**: Run `sol auth:info` to verify auth status
5. **Use --quiet for scripts**: Suppress progress messages with `-q`
