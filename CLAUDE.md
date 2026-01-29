# Dependency Security Policy

## Before Any Dependency Change

Run `/socket` before:
- Adding a new dependency
- Updating an existing dependency
- Changing package.json, requirements.txt, pyproject.toml, or any lockfile

## Required Workflow

1. **Run the check**: `/socket <package-name>` for new packages, or `/socket --file package.json` for bulk changes
2. **Evaluate results**: Review the output for security issues
3. **Document findings**: Include `/socket` output summary in your commit message or PR description

## Response to Findings

### Critical Issues
- **Do not proceed** with the dependency change
- Report the finding to the user
- Suggest safer alternatives if available

### Moderate Issues
- **Pause and explain** the risks to the user
- Proceed only with explicit user approval
- Document the accepted risk in the commit/PR

### Clean Results
- Proceed with the change
- Include "SOCKET_CHECK: clean" in commit message or PR description

## Output Format

When documenting `/socket` results, use this format:

```
SOCKET_CHECK: <status>
Package: <name>@<version>
Issues: <count critical>, <count moderate>
Details: <brief summary or "none">
```

## Exceptions

Skip `/socket` only when:
- The user explicitly requests it ("skip the socket check")
- The change is removing a dependency (not adding or updating)

In all other cases, running `/socket` is mandatory before dependency modifications.
