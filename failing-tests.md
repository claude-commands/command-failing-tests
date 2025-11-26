---
argument-hint: "[test-pattern]"
description: "Debug and fix failing tests"
model: claude-opus-4-5-20251101
allowed-tools: ["Bash", "Read", "Write", "Edit", "Glob", "Grep", "AskUserQuestion"]
---

**If `$ARGUMENTS` is empty or not provided:**

Run all tests and analyze any failures.

**Usage:** `/failing-tests [test-pattern]`

**Examples:**
- `/failing-tests` - Run all tests, analyze failures
- `/failing-tests auth` - Run only auth-related tests
- `/failing-tests --flaky` - Identify and fix flaky tests
- `/failing-tests --fix` - Automatically fix simple failures

**Workflow:**
1. Run tests and capture failures
2. Parse error messages and stack traces
3. Analyze root causes
4. Suggest or apply fixes
5. Verify fixes pass

Proceed with running all tests.

---

**If `$ARGUMENTS` is provided:**

Run specified tests and analyze failures.

## Configuration

- **Pattern**: `$ARGUMENTS`
  - Test name/pattern to filter
  - `--flaky`: Focus on flaky test detection
  - `--fix`: Auto-fix simple failures

## Steps

1. **Detect Test Framework**

   ```bash
   # Check for test runners
   ls package.json go.mod pytest.ini Cargo.toml 2>/dev/null
   ```

   Identify:
   - **Node.js**: Jest, Vitest, Mocha
   - **Go**: go test
   - **Python**: pytest, unittest
   - **Rust**: cargo test

2. **Run Tests**

   **Node.js (Jest)**:
   ```bash
   npx jest --json --outputFile=test-results.json 2>&1 || true
   ```

   **Node.js (Vitest)**:
   ```bash
   npx vitest run --reporter=json 2>&1 || true
   ```

   **Go**:
   ```bash
   go test -v -json ./... 2>&1 | tee test-output.json || true
   ```

   **Python**:
   ```bash
   pytest -v --tb=short 2>&1 || true
   ```

   Capture both stdout and stderr for analysis.

3. **Parse Test Results**

   Extract for each failure:
   - Test name and file location
   - Error message
   - Stack trace
   - Expected vs actual values
   - Relevant assertion

   ```json
   {
     "test": "should authenticate user with valid credentials",
     "file": "src/auth/__tests__/login.test.ts",
     "line": 45,
     "error": "Expected: 200, Received: 401",
     "stack": "at Object.<anonymous> (login.test.ts:45:5)"
   }
   ```

4. **Analyze Root Causes**

   For each failure, investigate:

   **a. Assertion Failures**
   - Compare expected vs actual
   - Check if test expectations are correct
   - Check if implementation changed

   **b. Runtime Errors**
   - Null/undefined access
   - Missing dependencies
   - Configuration issues

   **c. Timeout Failures**
   - Async operations not awaited
   - External service delays
   - Resource exhaustion

   **d. Environment Issues**
   - Missing env variables
   - Database state
   - Mocked dependencies

5. **Categorize Failures**

   | Category | Example | Fix Type |
   |----------|---------|----------|
   | Assertion | Wrong expected value | Update test |
   | Implementation | Code bug | Fix source |
   | Environment | Missing env var | Fix setup |
   | Flaky | Race condition | Fix timing |
   | Outdated | API changed | Update test |

6. **Detect Flaky Tests (if --flaky)**

   Run tests multiple times to identify non-deterministic failures:

   ```bash
   # Run tests 5 times
   for i in {1..5}; do
     npx jest --json >> flaky-results.json 2>&1
   done
   ```

   Analyze:
   - Tests that pass/fail inconsistently
   - Timing-dependent assertions
   - Shared state between tests
   - Order-dependent tests

7. **Generate Analysis Report**

   ```markdown
   # Test Failure Analysis

   **Total Tests**: 150
   **Passing**: 142
   **Failing**: 8

   ## Failure Summary

   | Test | Error | Root Cause | Fix |
   |------|-------|------------|-----|
   | auth/login | 401 != 200 | Mock not setup | Update mock |
   | api/users | Timeout | Async leak | Add await |
   | db/query | Connection | Env missing | Add DB_URL |

   ## Detailed Analysis

   ### 1. auth/login.test.ts - "should authenticate user"

   **Error**:
   ```
   Expected: 200
   Received: 401
   ```

   **Root Cause**: Auth mock not returning expected token

   **Fix**:
   ```typescript
   // In test setup, add:
   mockAuthService.authenticate.mockResolvedValue({
     token: 'valid-token',
     user: mockUser
   });
   ```

   **File**: `src/auth/__tests__/login.test.ts:45`

   ---

   ### 2. api/users.test.ts - "should list users"

   **Error**:
   ```
   Timeout - Async callback was not invoked within 5000ms
   ```

   **Root Cause**: Missing await on async operation

   **Fix**:
   ```typescript
   // Change line 67 from:
   const result = userService.list();
   // To:
   const result = await userService.list();
   ```

   ---

   ## Flaky Tests Detected

   | Test | Pass Rate | Issue |
   |------|-----------|-------|
   | cache/ttl | 80% | Timing sensitive |
   | ws/connect | 60% | Race condition |

   ## Recommended Actions

   1. **Immediate**: Fix auth mock setup (3 tests affected)
   2. **Quick win**: Add missing awaits (2 tests)
   3. **Investigate**: Flaky cache tests need refactor
   ```

8. **Apply Fixes (if --fix)**

   For simple fixes that can be automated:

   **a. Update assertions**
   - Snapshot updates
   - Expected value corrections

   **b. Add missing awaits**
   ```typescript
   // Find and fix unawaited async calls
   ```

   **c. Fix mock setups**
   ```typescript
   // Add missing mock implementations
   ```

   **d. Environment fixes**
   ```bash
   # Add missing env vars to test setup
   ```

9. **Verify Fixes**

   ```bash
   # Re-run previously failing tests
   npx jest --testPathPattern="failing-test-name"
   ```

   Confirm:
   - All targeted failures now pass
   - No new failures introduced
   - Tests are deterministic

## Output Structure

```markdown
# Test Failure Analysis

## Summary
[Pass/fail counts, severity breakdown]

## Failures by Category
[Grouped by root cause type]

## Detailed Analysis
[Per-test breakdown with fixes]

## Flaky Tests
[Tests with non-deterministic behavior]

## Applied Fixes
[What was automatically fixed]

## Remaining Issues
[What needs manual intervention]
```

## Common Failure Patterns

| Pattern | Symptoms | Solution |
|---------|----------|----------|
| Stale snapshot | Snapshot mismatch | Update snapshots |
| Missing mock | Undefined is not a function | Add mock |
| Race condition | Intermittent failure | Use waitFor/retry |
| State leak | Test passes alone, fails in suite | Isolate state |
| Time dependency | Fails at certain times | Mock Date |

## Notes

- Run tests in isolation first to identify state leaks
- Check CI vs local environment differences
- Flaky tests should be fixed, not skipped
- Auto-fixes should be reviewed before committing
- Consider test parallelization issues
