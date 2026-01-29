# Overview

## Why Dependency Guardrails Matter

Modern applications depend on hundreds of third-party packages. Each dependency is a trust decision: you're running someone else's code in your application, your CI pipeline, and potentially your production environment.

AI coding assistants accelerate development but can introduce dependencies without security evaluation. When Claude suggests `npm install some-package`, it doesn't inherently know:

- Whether the package has known vulnerabilities
- Whether it's actively maintained
- Whether it has been typosquatted or compromised
- What its transitive dependencies look like

## The Attack Surface

Dependency-based attacks are common:

- **Typosquatting**: `lodahs` instead of `lodash`
- **Dependency confusion**: Private package names registered on public registries
- **Compromised maintainers**: Legitimate packages with malicious updates
- **Abandoned packages**: No security patches for known issues

## Defense in Depth

No single control catches everything. This repo implements multiple layers:

```
┌─────────────────────────────────────────────────────┐
│  Layer 1: CLAUDE.md                                 │
│  Policy instructions that guide Claude's behavior   │
├─────────────────────────────────────────────────────┤
│  Layer 2: /socket Skill                             │
│  On-demand scanning during development              │
├─────────────────────────────────────────────────────┤
│  Layer 3: Rules                                     │
│  Enforcement that /socket must run                  │
├─────────────────────────────────────────────────────┤
│  Layer 4: CI Backstop                               │
│  PR validation requiring documented checks          │
├─────────────────────────────────────────────────────┤
│  Layer 5: Socket Firewall                           │
│  Install-time blocking of risky packages            │
└─────────────────────────────────────────────────────┘
```

Each layer catches issues that might slip through the previous one.

## What This Repo Provides

1. **CLAUDE.md template**: Copy to your project to instruct Claude
2. **Skill specification**: Interface for implementing `/socket`
3. **Rule templates**: Enforcement rules for dependency checks
4. **CI workflow**: GitHub Actions backstop
5. **Example project**: Demonstrates the workflow in practice
