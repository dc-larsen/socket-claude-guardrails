# CI Backstop

## Purpose

The CI backstop is a GitHub Actions workflow that fails pull requests when:

1. Dependency files were modified
2. The PR description does not contain a `/socket` check summary

This catches cases where Claude (or a human) skips the dependency check.

## How It Works

```
PR opened/updated
       │
       ▼
┌──────────────────┐
│ Check for changes│
│ to dep files     │
└────────┬─────────┘
         │
    ┌────┴────┐
    │ Changes │
    │ found?  │
    └────┬────┘
         │
    Yes  │  No
    ▼    │   ▼
┌────────┴─────┐  ┌─────────┐
│Check PR body │  │  Pass   │
│for SOCKET_   │  └─────────┘
│CHECK marker  │
└──────┬───────┘
       │
  ┌────┴────┐
  │ Marker  │
  │ found?  │
  └────┬────┘
       │
  Yes  │  No
  ▼    │   ▼
┌──────┴───┐  ┌─────────────────────────────┐
│   Pass   │  │ Fail with instructions      │
└──────────┘  │ to run /socket              │
              └─────────────────────────────┘
```

## Workflow File

```yaml
# .github/workflows/dependency-guardrail.yml

name: Dependency Guardrail

on:
  pull_request:
    types: [opened, edited, synchronize]

jobs:
  check-socket:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Check for dependency file changes
        id: deps
        run: |
          # Get list of changed files
          CHANGED=$(git diff --name-only origin/${{ github.base_ref }}...HEAD)

          # Define dependency file patterns
          DEP_FILES="package.json package-lock.json yarn.lock pnpm-lock.yaml \
                     requirements.txt requirements-dev.txt pyproject.toml poetry.lock \
                     Pipfile Pipfile.lock setup.py setup.cfg \
                     Cargo.toml Cargo.lock go.mod go.sum Gemfile Gemfile.lock"

          # Check if any dependency files changed
          DEPS_CHANGED=false
          for file in $DEP_FILES; do
            if echo "$CHANGED" | grep -q "$file"; then
              DEPS_CHANGED=true
              echo "Dependency file changed: $file"
            fi
          done

          echo "deps_changed=$DEPS_CHANGED" >> $GITHUB_OUTPUT

      - name: Verify Socket check in PR description
        if: steps.deps.outputs.deps_changed == 'true'
        env:
          PR_BODY: ${{ github.event.pull_request.body }}
        run: |
          if echo "$PR_BODY" | grep -qE "SOCKET_CHECK:|/socket summary"; then
            echo "✓ Socket check found in PR description"
          else
            echo "::error::Dependency files were modified but no Socket check found."
            echo ""
            echo "This PR modifies dependency files. Please:"
            echo "1. Run /socket on the changed dependencies"
            echo "2. Add the results to your PR description"
            echo ""
            echo "Example format:"
            echo "SOCKET_CHECK: clean"
            echo "Package: lodash@4.17.21"
            echo "Issues: 0 critical, 0 moderate"
            exit 1
          fi
```

## Customization

### Check Commit Messages Instead

If you prefer documenting in commit messages:

```yaml
- name: Verify Socket check in commits
  if: steps.deps.outputs.deps_changed == 'true'
  run: |
    COMMITS=$(git log --format=%B origin/${{ github.base_ref }}...HEAD)
    if echo "$COMMITS" | grep -qE "SOCKET_CHECK:"; then
      echo "✓ Socket check found in commit history"
    else
      echo "::error::No Socket check found in commits"
      exit 1
    fi
```

### Require Clean Status

To block PRs with any issues:

```yaml
- name: Verify clean Socket check
  if: steps.deps.outputs.deps_changed == 'true'
  env:
    PR_BODY: ${{ github.event.pull_request.body }}
  run: |
    if echo "$PR_BODY" | grep -q "SOCKET_CHECK: clean"; then
      echo "✓ Clean Socket check found"
    elif echo "$PR_BODY" | grep -q "SOCKET_CHECK: moderate"; then
      echo "::warning::Moderate issues found. Review required."
    elif echo "$PR_BODY" | grep -q "SOCKET_CHECK: critical"; then
      echo "::error::Critical issues found. Cannot merge."
      exit 1
    else
      echo "::error::No Socket check found"
      exit 1
    fi
```

### Allow Exceptions

Add an escape hatch for emergencies:

```yaml
- name: Check for override
  id: override
  run: |
    if echo "${{ github.event.pull_request.body }}" | grep -q "SOCKET_OVERRIDE:"; then
      echo "override=true" >> $GITHUB_OUTPUT
      echo "::warning::Socket check overridden. Manual review required."
    else
      echo "override=false" >> $GITHUB_OUTPUT
    fi

- name: Verify Socket check
  if: steps.deps.outputs.deps_changed == 'true' && steps.override.outputs.override == 'false'
  # ... rest of check
```

## Limitations

### Not a Hard Block

This check validates documentation, not the actual scan results. Someone could write "SOCKET_CHECK: clean" without running the check. For hard enforcement:

- Use Socket's GitHub App for PR-level scanning
- Require signed attestations from your CI/CD pipeline
- Implement branch protection with required status checks

### Lockfile-Only Changes

Lockfile updates from `npm install` or `pip install` without manifest changes still trigger the check. This is intentional: transitive dependency updates can introduce vulnerabilities.

To allow lockfile-only updates without checks:

```yaml
# Only check if manifest files changed (not lockfiles)
MANIFEST_FILES="package.json requirements.txt pyproject.toml Cargo.toml go.mod Gemfile"
```

### Multiple Packages

The marker format assumes a single package check. For bulk changes, adapt the format:

```
SOCKET_CHECK: summary
Packages checked: 5
Results: 5 clean, 0 moderate, 0 critical
```

## Troubleshooting

### Check Always Fails

Verify the PR body contains the exact marker text. Common issues:

- Smart quotes instead of straight quotes
- Extra whitespace
- Marker in a comment block that gets stripped

### Check Always Passes

Verify `fetch-depth: 0` is set. Without full history, git diff may not detect changes correctly.
