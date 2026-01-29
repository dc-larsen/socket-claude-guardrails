# Socket + Claude Code Guardrails

Prevent AI coding assistants from introducing risky dependencies into your codebase.

## The Problem

AI coding tools like Claude Code can add dependencies without evaluating their security posture. A single `npm install` suggestion might pull in:

- Packages with known vulnerabilities
- Typosquatted or malicious packages
- Abandoned projects with no maintenance
- Dependencies with problematic transitive dependencies

## The Solution: Defense in Depth

This repo provides a layered approach to dependency guardrails:

| Layer | Purpose | When it runs |
|-------|---------|--------------|
| **CLAUDE.md** | Instructs Claude to check dependencies before adding them | During Claude session |
| **Skill (`/socket`)** | Provides a command interface for dependency scanning | On-demand during development |
| **Rules** | Enforces that `/socket` must run before dependency changes | During Claude session |
| **CI Backstop** | Fails PRs that change dependencies without `/socket` output | On pull request |
| **Socket Firewall** | Blocks risky packages at install time | During `npm install` / `pip install` |

Each layer catches what the previous layer might miss.

## Quickstart

### 1. Copy CLAUDE.md to your project

```bash
curl -o CLAUDE.md https://raw.githubusercontent.com/dc-larsen/socket-claude-guardrails/main/CLAUDE.md
```

This tells Claude to run `/socket` before any dependency changes.

### 2. Implement the `/socket` skill (or use Socket CLI directly)

See [docs/socket-skill-spec.md](docs/socket-skill-spec.md) for the interface specification.

If you have the [Socket CLI](https://docs.socket.dev/docs/socket-cli) installed, you can use it directly:

```bash
socket npm info <package>
```

### 3. Add the CI backstop (optional)

Copy `.github/workflows/dependency-guardrail.yml` to your repo. This fails PRs that modify dependency files without documenting `/socket` results.

### 4. Enable Socket Firewall (recommended)

Install the [Socket GitHub App](https://github.com/apps/socket-security) or configure the [Socket CLI](https://docs.socket.dev/docs/socket-cli) to block risky packages at install time.

## Documentation

- [Overview](docs/overview.md) - Why dependency guardrails matter
- [Setup Guide](docs/setup.md) - Step-by-step configuration
- [CLAUDE.md Reference](docs/claude-md.md) - Policy file details
- [Rules Reference](docs/claude-rules.md) - Enforcement rules
- [Skill Specification](docs/socket-skill-spec.md) - `/socket` command interface
- [CI Backstop](docs/ci-backstop.md) - GitHub Actions configuration

## Example Project

See [examples/node](examples/node) for a minimal Node.js project demonstrating the workflow.

## License

MIT
