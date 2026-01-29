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

The skill works best with the Socket CLI installed:

```bash
npm install -g @socketsecurity/cli
```

Verify it works:

```bash
socket npm info lodash
```

### Alternative: Manual Checks

If you prefer not to install the CLI, the skill will guide you to check packages at [socket.dev](https://socket.dev) manually.

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
