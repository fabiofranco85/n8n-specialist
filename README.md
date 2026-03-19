# n8n Specialist Skill

A Claude Code skill for managing n8n workflows, executions, credentials, tags, and variables via the n8n REST API.

## Setup

### 1. Generate an n8n API key

1. Open your n8n Cloud instance
2. Go to **Settings > API**
3. Click **Create API Key**
4. Copy the key (it's only shown once)

### 2. Set environment variables

Add these to your shell profile (`~/.zshrc`, `~/.bashrc`, etc.):

```bash
export N8N_API_KEY="your-api-key-here"
export N8N_BASE_URL="https://your-instance.app.n8n.cloud"
```

Then reload:

```bash
source ~/.zshrc
```

Or set them in your Claude Code project's `.env` file.

### 3. Install the skill

Copy this skill directory into your Claude Code skills path, or reference it from your `CLAUDE.md`:

```markdown
# Skills
- /path/to/n8n-specialist
```

## What it does

- **Workflow management** — create, update, activate, deactivate, delete, export, and import workflows
- **Execution monitoring** — list, inspect, and retry workflow executions
- **Credential management** — create, update, delete credentials (secrets are never exposed)
- **Tag management** — organize workflows with tags
- **Variable management** — manage environment variables in n8n
- **User management** — list, create, and manage n8n users
- **Debugging** — inspect failed executions, identify failing nodes, and fix workflows
- **Documentation lookup** — searches n8n official docs when it needs info about nodes or configuration

## Example prompts

- "List all my active n8n workflows"
- "Create a new workflow that triggers on a webhook and sends a Slack message"
- "Why did workflow 1234 fail? Fix it and retry"
- "Deactivate all workflows tagged 'staging'"
- "Export all my workflows to a backup file"
- "What nodes are available for sending emails in n8n?"

## File structure

```
n8n-specialist/
├── SKILL.md              # Main skill instructions
├── README.md             # This file
└── references/
    └── api-endpoints.md  # Complete API endpoint reference
```
