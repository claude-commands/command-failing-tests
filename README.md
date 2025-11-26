# command-failing-tests

A Claude Code slash command for debugging and fixing failing tests.

## Installation

```bash
# Clone to your preferred location
git clone git@github.com:claude-commands/command-failing-tests.git <clone-path>/command-failing-tests

# Symlink (use full path to cloned repo)
ln -s <clone-path>/command-failing-tests/failing-tests.md ~/.claude/commands/failing-tests.md
```

## Usage

```text
/failing-tests              # Run all tests, analyze failures
/failing-tests auth         # Run only auth-related tests
/failing-tests --flaky      # Identify flaky tests
/failing-tests --fix        # Auto-fix simple failures
```

## What it does

1. Runs tests and captures failures
2. Parses error messages and stack traces
3. Analyzes root causes
4. Categorizes by fix type
5. Suggests or applies fixes
6. Verifies fixes pass

## Output Format

```markdown
# Test Failure Analysis

## Summary
- Total: 150 | Passing: 142 | Failing: 8

## Failures
| Test | Error | Root Cause |
|------|-------|------------|
| auth/login | 401 != 200 | Mock not setup |

## Detailed Analysis
### auth/login.test.ts
**Error**: Expected 200, Received 401
**Fix**: Update mock to return valid token
```

## Failure Categories

| Category | Example | Fix Type |
|----------|---------|----------|
| Assertion | Wrong expected value | Update test |
| Implementation | Code bug | Fix source |
| Environment | Missing env var | Fix setup |
| Flaky | Race condition | Fix timing |

## Framework Support

| Framework | Command | Notes |
|-----------|---------|-------|
| Jest | npx jest | --json output |
| Vitest | npx vitest | --reporter=json |
| Go | go test | -json flag |
| pytest | pytest | --tb=short |

## Requirements

- Git repository with tests
- Test framework installed
- Claude Code with Opus 4.5 model access

## Updates

```bash
cd <clone-path>/command-failing-tests && git pull
```
