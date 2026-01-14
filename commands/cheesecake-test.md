# /cheesecake test Command
# Purpose: Run CheeseCake test suites and individual tests
# Part of: CheeseCake v0.0.2 - Module 13 (Testing Framework)
#
# This command discovers and executes tests in .cheesecake files.
#
# Usage:
#   /cheesecake test [file] [options]
#
# Dependencies:
#   - skills/cheesecake/testing.md (testing spec)
#   - skills/cheesecake/vm.md (execution semantics)

---
name: cheesecake-test
description: Run CheeseCake tests with mocking and assertions
arguments: "[file] [options]"
---

# CheeseCake Test Runner

## Overview

The `/cheesecake test` command discovers and runs tests defined in CheeseCake files. Tests can use MOCK to avoid real AI calls and ASSERT to verify outcomes.

## Syntax

```bash
/cheesecake test [file] [options]
```

## Arguments

| Argument | Description |
|----------|-------------|
| `[file]` | Path to test file (optional - runs all if omitted) |

## Options

| Option | Description |
|--------|-------------|
| `--suite "name"` | Run only tests in specified suite |
| `--test "name"` | Run only the specified test |
| `--verbose` | Show detailed execution output |
| `--fail-fast` | Stop on first test failure |
| `--report <format>` | Output format: text, json, junit |
| `--strict` | Fail tests with unmocked sessions |

## Examples

### Run All Tests

```bash
/cheesecake test
```

Discovers and runs all TEST and TEST SUITE blocks in the current directory.

### Run Specific File

```bash
/cheesecake test tests/workflow-tests.cheesecake
```

Runs all tests in the specified file.

### Run Specific Suite

```bash
/cheesecake test --suite "Research Workflow Tests"
```

Runs only tests within the named suite.

### Run Specific Test

```bash
/cheesecake test --test "handles empty input"
```

Runs only the test with the matching name.

### Verbose Output

```bash
/cheesecake test --verbose
```

Shows detailed execution:
- Each MOCK registration
- Each statement executed
- Each ASSERT evaluation
- AI confidence for semantic assertions

### Fail Fast

```bash
/cheesecake test --fail-fast
```

Stops execution on the first failed test.

### Strict Mode

```bash
/cheesecake test --strict
```

Fails any test that tries to run an unmocked session.

## Output Formats

### Text (Default)

```
Running tests: workflow-tests.cheesecake

TEST SUITE: Research Workflow
  ✓ returns valid data (0.02s)
  ✓ handles empty input (0.01s)
  ✗ validates sources (0.03s)
    FAILED: ASSERT sources.length >= 3
    Expected: >= 3
    Actual: 2
    at line 34

TEST SUITE: Writer Workflow
  ✓ creates markdown (0.02s)

─────────────────────────────────
Tests:  4 total, 3 passed, 1 failed
Time:   0.08s
```

### JSON

```bash
/cheesecake test --report json
```

```json
{
  "summary": {
    "total": 4,
    "passed": 3,
    "failed": 1,
    "skipped": 0,
    "duration": 0.08
  },
  "suites": [
    {
      "name": "Research Workflow",
      "tests": [
        {
          "name": "returns valid data",
          "status": "pass",
          "duration": 0.02
        },
        {
          "name": "validates sources",
          "status": "fail",
          "duration": 0.03,
          "failure": {
            "assertion": "ASSERT sources.length >= 3",
            "expected": ">= 3",
            "actual": "2",
            "line": 34
          }
        }
      ]
    }
  ]
}
```

### JUnit XML

```bash
/cheesecake test --report junit
```

Compatible with CI/CD systems.

## Execution Flow

```
1. Parse Arguments
   - Determine target file(s)
   - Parse options

2. Discover Tests
   - Scan file(s) for TEST SUITE and TEST blocks
   - Build test registry
   - Report: "Found X suites, Y tests"

3. Execute Tests
   For each suite/test:
   a. Create isolated scope
   b. Run SETUP (if in suite)
   c. Execute test body
      - Register MOCKs
      - Execute statements
      - Resolve mocks for RUN SESSION
      - Evaluate ASSERTs
   d. Run TEARDOWN (if in suite)
   e. Record result

4. Report Results
   - Show pass/fail for each test
   - Show failure details
   - Print summary
```

## Mock Behavior

During test execution:

1. When `MOCK Agent RETURNS value` is encountered:
   - Register mock in test's mock registry

2. When `RUN SESSION(agent)` is encountered:
   - Check mock registry
   - If mocked: return mock value (no AI call)
   - If not mocked:
     - Strict mode: fail test
     - Normal mode: run real session (with warning)

## Assert Behavior

1. **Literal assertions**: Direct evaluation
   ```cheesecake
   ASSERT x == 5        # Compare values
   ASSERT x IS NOT NULL # Check for null
   ASSERT list CONTAINS item
   ```

2. **Semantic assertions**: AI evaluation
   ```cheesecake
   ASSERT **{result} is well-structured**
   ```
   - AI evaluates the condition
   - Returns TRUE/FALSE
   - Shows confidence in verbose mode

## Error Handling

| Error | Behavior |
|-------|----------|
| Parse error | Show error, skip file |
| ASSERT failure | Mark test FAILED, continue to next |
| Uncaught exception | Mark test ERROR, continue to next |
| Unmocked session (strict) | Mark test FAILED |

## Integration

### With CI/CD

```yaml
# Example GitHub Actions
- name: Run CheeseCake Tests
  run: |
    /cheesecake test --report junit > test-results.xml
```

### With Cost Management

```cheesecake
CONFIG:
  BUDGET: $0.00  # Enforce all mocks
END CONFIG

TEST SUITE "Zero Cost Tests":
  # Any unmocked session would exceed budget
END TEST SUITE
```

## Best Practices

1. **Mock all external calls**
   - Avoids costs
   - Ensures reproducibility

2. **Use descriptive test names**
   - Makes failures easy to understand

3. **One concept per test**
   - Easier to debug failures

4. **Use semantic assertions sparingly**
   - They require AI evaluation
   - Best for quality checks that can't be literal

5. **Organize with suites**
   - Group related tests
   - Share setup/teardown

## Related Commands

- `/cheesecake run` - Execute workflows
- `/cheesecake validate` - Validate syntax
- `/cheesecake explain` - Explain file structure

## See Also

- `skills/cheesecake/testing.md` - Full testing specification
- `skills/cheesecake/vm.md` - Test execution semantics
