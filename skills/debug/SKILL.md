---
name: debug
description: "Analyzes session state and diagnostics to help troubleshoot issues. Use /debug or /debug <specific question> for focused analysis."
context: fork
disable-model-invocation: true
user-invocable: true
model_role: general
---

# /debug — Session Diagnostics

Analyze session state and environment to help troubleshoot issues.

Specific question or focus area: `$ARGUMENTS`

---

## Step 1 — Gather Diagnostics

Collect the following diagnostic information:

### Environment Variables
```bash
# Key environment variables (redact sensitive values like API keys)
env | grep -E "^(PATH|HOME|USER|SHELL|PWD|AMPLIFIER|PYTHONPATH|VIRTUAL_ENV|NODE_ENV|CI)" | sort
```

### Working Directory
```bash
pwd
ls -la
```

### Git State
```bash
# Current branch, status, and recent commits
git branch --show-current 2>/dev/null || echo "Not a git repo"
git status --short 2>/dev/null
git log --oneline -5 2>/dev/null
```

### Python Environment
```bash
# Python version and installed packages
python --version 2>/dev/null || python3 --version 2>/dev/null
pip list 2>/dev/null | head -20 || echo "pip not available"
which python 2>/dev/null || which python3 2>/dev/null
```

### Recent Errors
Check for recent error output in:
- Session logs or error files in the current directory
- Any `.log` files or `error.log` files
- Stack traces or tracebacks in recent command output

```bash
find . -maxdepth 2 -name "*.log" -newer . 2>/dev/null | head -5
```

---

## Step 2 — Analyze

If `$ARGUMENTS` is provided, perform a **focused analysis** on the specific question: `$ARGUMENTS`

- Investigate the specific issue or question directly
- Look for relevant configuration, code, or state related to the question
- Identify the most likely root cause based on gathered diagnostics

If no arguments are provided, perform a **general health report**:

- **Session configuration**: Review available tools, active model settings, and session parameters
- **Available tools**: List tools currently accessible and note any that appear missing or misconfigured
- **Issues detected**: Identify any anomalies in the gathered diagnostics (missing dependencies, wrong Python version, unexpected env vars, dirty git state, etc.)
- **Suggestions**: Recommend concrete next steps based on findings

---

## Step 3 — Report

Present findings in the following structured format:

```
## Debug Report

### Environment
- Working directory: <path>
- Python: <version> at <path>
- Git branch: <branch> (<N uncommitted changes if any>)
- Virtual env: <path or "none">
- Key env vars: <relevant findings>

### Configuration
- <List session configuration details>
- <Available tool categories and count>
- <Any relevant settings or parameters>

### Issues Found
- <Issue 1>: <description and evidence>
- <Issue 2>: <description and evidence>
- (or "No issues detected" if clean)

### Recommendations
1. <Actionable recommendation 1>
2. <Actionable recommendation 2>
3. <Actionable recommendation 3>
```

If `$ARGUMENTS` was provided, prefix the report with:
```
## Focused Debug: $ARGUMENTS
```
and ensure the report directly addresses the specific question before the general sections.
