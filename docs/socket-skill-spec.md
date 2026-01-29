# /socket Skill Specification

## Overview

The `/socket` skill provides a command-line interface for checking package security before installation. This document defines the interface; implementation varies by environment.

## Command Syntax

```
/socket [options] [package...]
```

### Arguments

| Argument | Description |
|----------|-------------|
| `package` | Package name(s) to check. Format: `name` or `name@version` |

### Options

| Option | Description |
|--------|-------------|
| `--npm` | Check npm registry (default for .js/.ts projects) |
| `--pypi` | Check PyPI registry (default for .py projects) |
| `--all` | Check all supported registries |
| `--direct-only` | Check only the specified package (default) |
| `--direct-and-transitive` | Check package and all transitive dependencies |
| `--file <path>` | Check all dependencies in a manifest file |
| `--json` | Output in JSON format |

## Examples

### Check a Single Package

```
/socket lodash
```

Output:
```
SOCKET_CHECK: clean
Package: lodash@4.17.21
Issues: 0 critical, 0 moderate
Details: none
```

### Check with Transitive Dependencies

```
/socket express --direct-and-transitive
```

Output:
```
SOCKET_CHECK: moderate
Package: express@4.18.2
Issues: 0 critical, 2 moderate

Moderate Issues:
- cookie@0.5.0: Outdated dependency (2 years since last update)
- qs@6.11.0: Known prototype pollution in older versions

Recommendation: Update to express@4.19.0 which addresses these issues.
```

### Check a Manifest File

```
/socket --file package.json
```

Output:
```
SOCKET_CHECK: critical
File: package.json
Packages checked: 24
Issues: 1 critical, 5 moderate

Critical Issues:
- event-stream@3.3.6: Known malicious package (flatmap-stream incident)

Moderate Issues:
- moment@2.29.1: Deprecated, consider dayjs or date-fns
- request@2.88.2: Deprecated, consider axios or node-fetch
- lodash@4.17.15: Update available with security fixes
- colors@1.4.0: Known sabotaged version
- faker@5.5.3: Abandoned after maintainer protest

Action Required: Remove event-stream before proceeding.
```

### Check Python Package

```
/socket --pypi requests
```

Output:
```
SOCKET_CHECK: clean
Package: requests@2.31.0
Registry: PyPI
Issues: 0 critical, 0 moderate
Details: none
```

## Output Format

### Status Values

| Status | Meaning |
|--------|---------|
| `clean` | No issues found |
| `moderate` | Issues found but not blocking |
| `critical` | Blocking issues, do not proceed |

### Standard Output

```
SOCKET_CHECK: <status>
Package: <name>@<version>
Issues: <N> critical, <N> moderate
Details: <summary or "none">
```

### With Issues

```
SOCKET_CHECK: <status>
Package: <name>@<version>
Issues: <N> critical, <N> moderate

Critical Issues:
- <package>: <description>

Moderate Issues:
- <package>: <description>

Recommendation: <action>
```

### JSON Output

```json
{
  "status": "moderate",
  "package": "express",
  "version": "4.18.2",
  "issues": {
    "critical": 0,
    "moderate": 2
  },
  "details": [
    {
      "severity": "moderate",
      "package": "cookie",
      "version": "0.5.0",
      "issue": "Outdated dependency"
    }
  ],
  "recommendation": "Update to express@4.19.0"
}
```

## Issue Types

The skill should detect and report:

| Category | Examples |
|----------|----------|
| **Security** | Known vulnerabilities (CVEs), malicious code |
| **Supply Chain** | Typosquatting, dependency confusion, compromised maintainers |
| **Quality** | Deprecated packages, unmaintained projects, unstable versions |
| **License** | Incompatible licenses, missing license info |

## Implementation Notes

### v1 Limitations

This specification describes a reporting-only interface:

- Does not auto-fix issues
- Does not automatically update packages
- Does not modify dependency files

The skill reports findings and suggests alternatives. The user decides how to proceed.

### Integration with Socket

If using Socket's infrastructure:

```bash
# Direct CLI usage
socket npm info <package>
socket npm info <package> --json

# API usage
curl -H "Authorization: Bearer $SOCKET_API_KEY" \
  "https://api.socket.dev/v0/npm/package-info/<package>"
```

### Custom Implementation

Build your own by:

1. Querying package registry metadata
2. Cross-referencing vulnerability databases (NVD, GitHub Advisory)
3. Checking Socket's public package scores
4. Formatting output per this spec

## Error Handling

```
/socket nonexistent-package-12345
```

Output:
```
SOCKET_CHECK: error
Package: nonexistent-package-12345
Error: Package not found in npm registry
```

```
/socket lodash --pypi
```

Output:
```
SOCKET_CHECK: error
Package: lodash
Error: Package not found in PyPI registry. Did you mean --npm?
```
