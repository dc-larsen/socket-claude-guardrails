# Setup Guide

## Prerequisites

- A project using npm, pip, or another supported package manager
- Claude Code or another AI coding assistant
- (Optional) Socket CLI or Socket GitHub App

## Step 1: Add CLAUDE.md

Copy the CLAUDE.md from this repo to your project root:

```bash
curl -o CLAUDE.md https://raw.githubusercontent.com/dc-larsen/socket-claude-guardrails/main/CLAUDE.md
```

Or copy manually from [CLAUDE.md](../CLAUDE.md).

This file tells Claude to run `/socket` before any dependency change.

## Step 2: Install the /socket Skill

Copy the skill to your Claude configuration:

```bash
mkdir -p ~/.claude/skills/socket
curl -o ~/.claude/skills/socket/SKILL.md \
  https://raw.githubusercontent.com/dc-larsen/socket-claude-guardrails/main/.claude/skills/socket/SKILL.md
```

Or for project-specific installation:

```bash
mkdir -p .claude/skills/socket
curl -o .claude/skills/socket/SKILL.md \
  https://raw.githubusercontent.com/dc-larsen/socket-claude-guardrails/main/.claude/skills/socket/SKILL.md
```

### Install Socket CLI (Recommended)

The skill works best with the Socket CLI installed and authenticated.

**1. Install the CLI:**
```bash
npm install -g @socketsecurity/cli
```

**2. Get an API key:**
- Sign up or log in at [socket.dev](https://socket.dev)
- Go to Settings > API Tokens
- Generate a new token

**3. Authenticate:**
```bash
export SOCKET_API_KEY=sktsec_your_token_here
```

Or add to your shell profile (`~/.zshrc` or `~/.bashrc`):
```bash
echo 'export SOCKET_API_KEY=sktsec_your_token_here' >> ~/.zshrc
```

**4. Verify it works:**
```bash
socket npm/lodash
```

You should see your org name in the banner (e.g., `org: your-org-name`).

### Alternative: Manual Checks

Without the CLI, you can check packages manually at [socket.dev](https://socket.dev). The skill will provide direct links when CLI access is unavailable.

## Step 3: Add Claude Rules (Optional)

If your Claude setup supports rules, add enforcement rules from [claude-rules.md](claude-rules.md). This ensures Claude won't skip the `/socket` step.

## Step 4: Add CI Backstop (Optional)

Copy the GitHub Actions workflow:

```bash
mkdir -p .github/workflows
curl -o .github/workflows/dependency-guardrail.yml \
  https://raw.githubusercontent.com/dc-larsen/socket-claude-guardrails/main/.github/workflows/dependency-guardrail.yml
```

This fails PRs that modify dependency files without documenting `/socket` results.

## Step 5: Enable Socket Firewall (Recommended)

For install-time protection:

1. Install the [Socket GitHub App](https://github.com/apps/socket-security) for PR scanning
2. Configure Socket CLI for local install blocking

See [Socket documentation](https://docs.socket.dev) for details.

## Verification

Test your setup:

1. Start a Claude session in your project
2. Ask Claude to add a dependency: "Add lodash to the project"
3. Verify Claude runs `/socket lodash` before making changes
4. Check that the commit message or PR includes the `/socket` output

If Claude adds the dependency without checking, review your CLAUDE.md placement and content.
