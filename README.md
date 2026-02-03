# Sol Skill

Claude Code skill for [Sol](https://github.com/menor/sol), an agent-optimized CLI for Upsun.

## What is Sol?

Sol is a CLI designed for AI agents to manage Upsun projects. It features:

- **Structured output**: JSON by default, TOON for token efficiency
- **Command schemas**: `--schema` flag for programmatic command discovery
- **Composable commands**: Each command does one thing well

## Installation

### 1. Install Sol CLI

Download the latest release from the [Sol releases page](https://github.com/menor/sol/releases) or build from source:

```bash
git clone https://github.com/menor/sol.git
cd sol
go build -o sol .
mv sol /usr/local/bin/
```

### 2. Authenticate

```bash
sol auth:login
```

### 3. Add the Skill to Claude Code

Add to your Claude Code settings:

```json
{
  "skills": [
    "github:menor/sol-skill"
  ]
}
```

Or clone locally and reference the path.

## Usage

Once installed, Claude Code can use Sol commands to help you manage Upsun:

- "List my Upsun projects"
- "Show environments for project xyz"
- "Set the DATABASE_URL variable on staging"
- "Redeploy the production environment"
- "SSH into the web container"

## License

Copyright (c) 2026 Jose Menor. All rights reserved.
