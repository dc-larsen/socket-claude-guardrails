# Claude Rules Reference

## Overview

Rules provide enforcement beyond CLAUDE.md guidance. While CLAUDE.md instructs Claude what to do, rules can enforce that certain conditions are met.

## Copy/Paste Rules

### Rule 1: Dependency Check Required

```
When modifying any dependency file (package.json, requirements.txt, pyproject.toml,
Cargo.toml, go.mod, Gemfile, or their lockfiles):

1. Run /socket to check the dependency before making changes
2. Include the /socket output in your response
3. Do not proceed with critical findings unless the user explicitly approves
```

### Rule 2: Document All Dependency Changes

```
Every commit or PR that modifies dependency files must include a SOCKET_CHECK
marker in the commit message or PR description.

Format:
SOCKET_CHECK: <clean|moderate|critical>
Package: <name>@<version>
Issues: <summary>
```

### Rule 3: No Silent Dependency Updates

```
Never update dependencies without informing the user of:
1. What is being updated
2. The /socket check results
3. Any breaking changes or security implications

If /socket is unavailable, inform the user and ask how to proceed.
```

## Implementation

How you implement these rules depends on your Claude setup:

### Claude Code Projects

Add rules to your project configuration or include them in CLAUDE.md under a "Rules" section.

### Custom Integrations

If you're building a custom Claude integration, enforce rules in your application logic by:

1. Detecting dependency file modifications in Claude's proposed changes
2. Checking for `/socket` execution in the conversation history
3. Validating the output format before allowing commits

### MCP Server

Build an MCP server that intercepts file write operations and validates `/socket` was run:

```python
# Pseudocode
def validate_write(file_path, content, conversation_history):
    if is_dependency_file(file_path):
        if not socket_check_in_history(conversation_history):
            raise ValidationError("Run /socket before modifying dependencies")
```

## Adapting Rules

### For Stricter Environments

```
All dependency changes require:
1. /socket check with clean or moderate result
2. Security team approval for any moderate findings
3. No critical findings permitted under any circumstances
```

### For Development Speed

```
/socket is required for:
- Production dependencies only (not devDependencies)
- New packages only (not updates to existing packages)
- Packages from unknown publishers
```

### For Specific Package Managers

```
For npm packages:
- Run: socket npm info <package>
- Check: npm audit for additional context

For Python packages:
- Run: socket pypi info <package>
- Check: pip-audit for additional context
```

## Limitations

Rules in CLAUDE.md are guidance, not hard enforcement. Claude may occasionally skip steps, especially in complex conversations. The CI backstop provides a harder guarantee that checks were documented.
