# CLAUDE.md Reference

## Purpose

CLAUDE.md is a project-level configuration file that provides instructions to Claude Code. It's read at the start of each session and influences Claude's behavior throughout.

## Location

Place CLAUDE.md in your project root:

```
my-project/
├── CLAUDE.md          # Claude reads this
├── package.json
├── src/
└── ...
```

## Key Sections

### Before Any Dependency Change

Defines the trigger conditions. Claude will check for dependency changes before modifying:

- package.json
- package-lock.json
- requirements.txt
- pyproject.toml
- poetry.lock
- Any other dependency manifest

### Required Workflow

The three-step process:

1. **Run the check**: Execute `/socket` with appropriate arguments
2. **Evaluate results**: Parse the output for issues
3. **Document findings**: Include results in commit/PR

### Response to Findings

Defines behavior based on severity:

| Severity | Action |
|----------|--------|
| Critical | Block the change, suggest alternatives |
| Moderate | Warn user, require explicit approval |
| Clean | Proceed normally |

### Output Format

Standardized format for documenting results:

```
SOCKET_CHECK: clean
Package: lodash@4.17.21
Issues: 0 critical, 0 moderate
Details: none
```

This format is parsed by the CI backstop to verify checks were run.

## Customization

### Adjusting Severity Thresholds

Modify the "Response to Findings" section to match your risk tolerance:

```markdown
### Critical Issues
- **Warn the user** but allow override with explicit approval
```

### Adding Package Managers

Extend the trigger conditions:

```markdown
Run `/socket` before:
- Adding a new dependency
- Changing Cargo.toml, go.mod, Gemfile, or any lockfile
```

### Requiring Approval for All Changes

For high-security environments:

```markdown
### All Dependency Changes
- Require explicit user approval before any dependency modification
- Document approval in commit message
```

## Troubleshooting

### Claude Ignores CLAUDE.md

- Verify the file is in the project root
- Check for syntax errors in the Markdown
- Ensure the file is not in .gitignore (Claude may skip ignored files)

### Claude Runs /socket But Doesn't Wait

Add explicit instruction:

```markdown
**Wait for /socket to complete** before making any changes. Do not proceed until you have the full output.
```

### Too Many False Positives

Adjust your `/socket` implementation to filter noise, or modify the severity thresholds in CLAUDE.md.
