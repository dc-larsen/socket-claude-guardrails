# Socket Dependency Check

Check npm or PyPI packages for security issues before installation.

## Usage

```
/socket <package>           # Check a single package
/socket <pkg1> <pkg2>       # Check multiple packages
/socket --file package.json # Check all dependencies in a file
/socket --pypi <package>    # Check PyPI package (default is npm)
```

## Instructions

<skill>
name: socket
description: Check package security before adding dependencies
user-invocable: true
arguments:
  - name: packages
    description: Package name(s) or --file <path>
    required: false
</skill>

When the user invokes `/socket`, follow these steps:

### 1. Parse Arguments

Determine what to check:
- If `--file <path>` is provided, read the manifest and extract all dependencies
- If `--pypi` flag is present, use PyPI registry
- Otherwise, treat arguments as npm package names
- If no arguments, check the project's package.json or requirements.txt

### 2. Run Socket CLI (requires auth)

The Socket CLI requires authentication. Users must:
1. Have the CLI installed: `npm install -g @socketsecurity/cli`
2. Have a Socket account and API key
3. Set the `SOCKET_API_KEY` environment variable or run `socket login`

Check a package:
```bash
socket npm/lodash              # Latest version
socket npm/lodash@4.17.21      # Specific version
socket npm/lodash --markdown   # Detailed report
```

For PyPI:
```bash
socket pypi/requests
```

### 3. Fallback (no CLI or no auth)

If the Socket CLI is unavailable or not authenticated:

1. **Direct user to socket.dev**: Ask them to check manually at:
   - npm: https://socket.dev/npm/package/<package>
   - PyPI: https://socket.dev/pypi/package/<package>

2. **Use npm registry metadata** (limited info, no Socket scores):
   ```bash
   npm view <package> --json
   ```

3. **Recommend installing the CLI**:
   - Sign up at https://socket.dev
   - Generate an API key in dashboard
   - `export SOCKET_API_KEY=<your-key>`

### 4. Evaluate Results

Classify findings by severity:

**Critical** (block by default):
- Known malware or malicious code
- Typosquatting detection
- Install scripts that execute suspicious commands
- Network access during install
- Filesystem access outside package directory

**Moderate** (warn, allow with approval):
- Deprecated packages
- Unmaintained (>2 years since update)
- High CVE counts without patches
- Obfuscated code
- Shell access capabilities

**Clean**:
- No issues detected

### 5. Output Format

Always output in this exact format:

```
SOCKET_CHECK: <clean|moderate|critical>
Package: <name>@<version>
Issues: <N> critical, <N> moderate
Details: <brief summary or "none">
```

For multiple packages:
```
SOCKET_CHECK: summary
Packages checked: <N>
Results: <N> clean, <N> moderate, <N> critical

<package1>: clean
<package2>: moderate - deprecated, suggest alternative
<package3>: critical - known malware, do not install
```

### 6. Recommendations

For critical issues:
- State clearly: "Do not install this package"
- Suggest alternatives if known

For moderate issues:
- Explain the risk
- Ask user if they want to proceed
- Note that risk is being accepted if they continue

For clean results:
- Confirm it's safe to proceed
- Note the check was completed

### 7. Document for CI

Remind the user to include the SOCKET_CHECK output in their commit message or PR description so the CI backstop passes.

## Examples

### Single Package Check
```
User: /socket lodash

SOCKET_CHECK: clean
Package: lodash@4.17.21
Issues: 0 critical, 0 moderate
Details: none

This package looks safe to install.
```

### Risky Package
```
User: /socket event-stream

SOCKET_CHECK: critical
Package: event-stream@3.3.6
Issues: 1 critical, 0 moderate
Details: Known malicious code injection (flatmap-stream incident)

Do not install this package. It was compromised in November 2018 to steal cryptocurrency wallet credentials.

Alternatives:
- highland (functional streams)
- through2 (simple transforms)
- Node.js native stream.Transform
```

### File Check
```
User: /socket --file package.json

SOCKET_CHECK: summary
Packages checked: 12
Results: 10 clean, 2 moderate, 0 critical

express@4.18.2: clean
lodash@4.17.21: clean
moment@2.29.4: moderate - deprecated, consider dayjs
request@2.88.2: moderate - deprecated, consider axios
...

2 packages have moderate issues. Do you want to proceed with these dependencies?
```
