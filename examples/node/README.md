# Node.js Example Project

A minimal Express server demonstrating the Socket guardrails workflow.

## Setup

```bash
npm install
npm start
```

## Workflow Example

### Scenario: Adding a New Dependency

You want to add a date formatting library. Here's how the guardrails workflow should proceed.

#### Step 1: Request the Change

Tell Claude:
> "Add a library for date formatting"

#### Step 2: Claude Runs /socket

Before modifying package.json, Claude should run:

```
/socket dayjs
```

Expected output:
```
SOCKET_CHECK: clean
Package: dayjs@1.11.10
Issues: 0 critical, 0 moderate
Details: none
```

#### Step 3: Claude Makes the Change

With clean results, Claude updates package.json:

```json
{
  "dependencies": {
    "express": "^4.18.2",
    "dayjs": "^1.11.10"
  }
}
```

#### Step 4: Document in Commit

The commit message includes:

```
Add dayjs for date formatting

SOCKET_CHECK: clean
Package: dayjs@1.11.10
Issues: 0 critical, 0 moderate
```

### Scenario: Risky Dependency Detected

You ask Claude to add `event-stream` (a package with a known supply chain attack history).

#### Step 1: Claude Runs /socket

```
/socket event-stream
```

Output:
```
SOCKET_CHECK: critical
Package: event-stream@3.3.6
Issues: 1 critical, 0 moderate

Critical Issues:
- Known malicious code injection (flatmap-stream incident, Nov 2018)
- Package was compromised to steal cryptocurrency wallet credentials

Recommendation: Do not install. Consider alternatives:
- highland (functional streams)
- through2 (simple stream transforms)
- Node.js native stream.Transform
```

#### Step 2: Claude Blocks the Change

Following the CLAUDE.md policy, Claude refuses to add the dependency and explains the risk.

#### Step 3: Choose an Alternative

You ask for `highland` instead:

```
/socket highland
```

Output:
```
SOCKET_CHECK: clean
Package: highland@2.13.5
Issues: 0 critical, 0 moderate
Details: none
```

Claude proceeds with the safe alternative.

### Scenario: Moderate Issues

You want to add `moment` for date handling.

```
/socket moment
```

Output:
```
SOCKET_CHECK: moderate
Package: moment@2.29.4
Issues: 0 critical, 1 moderate

Moderate Issues:
- Deprecated: maintainers recommend alternatives for new projects

Recommendation: Consider dayjs or date-fns for new projects.
Use moment only if you need specific legacy features.
```

Claude explains the situation and asks for your decision:

> "The moment library works but is deprecated. The maintainers recommend dayjs or date-fns for new projects. Should I proceed with moment, or would you prefer an alternative?"

## PR Workflow

When submitting a PR that adds dependencies, include the /socket output in the PR description:

```markdown
## Changes

Added dayjs for date formatting in the user dashboard.

## Socket Check

SOCKET_CHECK: clean
Package: dayjs@1.11.10
Issues: 0 critical, 0 moderate
Details: none
```

The CI backstop will verify this marker is present before allowing the PR to merge.
