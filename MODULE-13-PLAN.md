# Module 13: Testing Framework - Implementation Plan

**Module**: Testing Framework (v0.0.2)
**Status**: In Progress
**Date**: 2026-01-09

---

## Overview

Module 13 adds a comprehensive testing framework to CheeseCake, enabling users to write tests for workflows with mocked agent responses, assertions, and organized test suites.

---

## Features

### 1. TEST SUITE Construct

Groups related tests with shared setup/teardown:

```cheesecake
TEST SUITE "Workflow Tests":
  SETUP:
    # Runs before each test
    VAR fixtures = LOAD "test-data.json"
  END SETUP

  TEARDOWN:
    # Runs after each test
    CLEANUP temp_files
  END TEARDOWN

  TEST "test case 1":
    # Test body
  END TEST

  TEST "test case 2":
    # Test body
  END TEST

END TEST SUITE
```

**Features:**
- Named suites for organization
- SETUP block runs before each test
- TEARDOWN block runs after each test
- Multiple tests per suite
- Nested suites (optional)

### 2. TEST Construct

Individual test cases:

```cheesecake
TEST "descriptive test name":
  # Arrange
  MOCK Agent RETURNS expected_value

  # Act
  VAR result = CALL workflow(input)

  # Assert
  ASSERT result IS NOT NULL
  ASSERT **{result} meets quality standards**
END TEST
```

**Features:**
- Descriptive string name
- Isolated scope per test
- Pass/fail reporting
- Error capture and display

### 3. MOCK Construct

Replace agent sessions with fixed responses:

```cheesecake
# Simple mock
MOCK Researcher RETURNS "Sample research data"

# Mock with structured data
MOCK Analyzer RETURNS {
  findings: ["item1", "item2"],
  confidence: 0.95,
  summary: "Analysis complete"
}

# Mock specific task patterns
MOCK Writer FOR TASK MATCHING "write article" RETURNS "Article content..."
MOCK Writer FOR TASK MATCHING "write summary" RETURNS "Summary content..."

# Mock to throw error
MOCK APIAgent THROWS "Connection timeout"
```

**Features:**
- Replace any agent with fixed response
- Structured return values
- Task-specific mocks
- Error simulation
- Scope: applies within current TEST block

### 4. ASSERT Construct

Verify conditions:

```cheesecake
# Literal assertions
ASSERT value IS NOT NULL
ASSERT value IS NULL
ASSERT value == expected
ASSERT value != unexpected
ASSERT value > minimum
ASSERT value < maximum
ASSERT value >= minimum
ASSERT value <= maximum
ASSERT list CONTAINS item
ASSERT string MATCHES "pattern"

# Semantic assertions (AI-evaluated)
ASSERT **{result} is well-structured**
ASSERT **{article} is publication-ready**
ASSERT **{response} answers the question**

# Custom failure message
ASSERT value > 0 MESSAGE "Value must be positive"
```

**Features:**
- Literal comparisons
- Semantic AI-evaluated conditions
- Custom failure messages
- Clear pass/fail output

### 5. Test Execution Command

```bash
# Run all tests
/cheesecake test

# Run specific file
/cheesecake test tests/my-tests.cheesecake

# Run specific suite
/cheesecake test --suite "Workflow Tests"

# Run specific test
/cheesecake test --test "handles empty input"

# Verbose output
/cheesecake test --verbose

# Stop on first failure
/cheesecake test --fail-fast

# Generate report
/cheesecake test --report junit
```

---

## Implementation Scope

### In Scope for v0.0.2
- TEST SUITE with SETUP/TEARDOWN
- TEST individual test cases
- MOCK for agent responses
- MOCK with task pattern matching
- MOCK THROWS for error simulation
- ASSERT literal conditions
- ASSERT semantic conditions
- /cheesecake test command
- Test result reporting
- Pass/fail summary

### Out of Scope (Future)
- Code coverage metrics
- Test parallelization
- Snapshot testing
- Performance benchmarks
- CI/CD integration helpers

---

## Files to Create/Modify

### 1. NEW: skills/cheesecake/testing.md (Target: 700+ lines)
Complete specification for testing framework:
- TEST SUITE syntax and semantics
- TEST syntax and semantics
- MOCK syntax and semantics
- ASSERT syntax and semantics
- Test execution protocol
- Examples and best practices

### 2. UPDATE: skills/cheesecake/SKILL.md (+180 lines)
Add new section 13: "Testing Framework (v0.0.2+)"
- TEST SUITE construct
- TEST construct
- MOCK construct
- ASSERT construct
- Rules and constraints

Renumber existing sections:
- 13 → 14 (Error Handling)
- 14 → 15 (Functions)
- ... and so on to 20

### 3. UPDATE: skills/cheesecake/vm.md (+250 lines)
Add new section 11: "Test Execution (v0.0.2+)"
- Test discovery protocol
- Test execution protocol
- Mock registry management
- Assertion evaluation
- Result reporting

Renumber existing sections:
- 11 → 12 (Error Handling)
- 12 → 13 (Context Passing)
- ... and so on to 19

### 4. NEW: commands/cheesecake-test.md (Target: 200+ lines)
Test command specification:
- Command syntax
- Options and flags
- Test discovery
- Output formats
- Examples

### 5. NEW: test-testing-framework.cheesecake (Target: 250+ lines)
Test file covering:
- TEST SUITE declaration
- SETUP/TEARDOWN
- TEST cases
- MOCK agent responses
- MOCK with patterns
- MOCK THROWS
- ASSERT literal
- ASSERT semantic
- Nested structures
- Error handling

### 6. UPDATE: CHANGELOG.md (+80 lines)
Document Module 13 completion

### 7. NEW: MODULE-13-COMPLETE.md
Completion marker with summary

---

## Syntax Specification

### TEST SUITE

```
TEST SUITE "suite name":
  [SETUP:
    # Setup statements
  END SETUP]

  [TEARDOWN:
    # Teardown statements
  END TEARDOWN]

  TEST "test name":
    # Test body
  END TEST

  [... more tests ...]

END TEST SUITE
```

**Rules:**
- Suite name must be unique string
- SETUP is optional, runs before each test
- TEARDOWN is optional, runs after each test
- At least one TEST required
- Tests execute in declaration order

### TEST

```
TEST "test name":
  # Test body with MOCKs and ASSERTs
END TEST
```

**Rules:**
- Test name must be unique within suite
- Isolated scope (variables don't leak)
- Can contain MOCK, ASSERT, any valid statements
- Fails on first failed ASSERT or uncaught error

### MOCK

```
MOCK AgentName RETURNS value
MOCK AgentName FOR TASK MATCHING "pattern" RETURNS value
MOCK AgentName THROWS "error message"
```

**Rules:**
- Applies within current TEST block only
- Multiple MOCKs for same agent allowed (task-specific)
- RETURNS can be any value type
- THROWS simulates session error
- Pattern uses glob matching

### ASSERT

```
ASSERT condition [MESSAGE "custom message"]
ASSERT **semantic condition** [MESSAGE "custom message"]
```

**Rules:**
- Literal conditions evaluated directly
- Semantic conditions (`**...**`) evaluated by AI
- MESSAGE is optional custom failure text
- Test fails immediately on failed ASSERT

---

## Execution Model

### Test Discovery

When running `/cheesecake test`:

1. Scan for TEST SUITE blocks in file(s)
2. Collect all TEST blocks (in suites or standalone)
3. Build test registry with metadata
4. Report discovered tests

### Test Execution Protocol

For each test:

```
1. Create isolated test scope
2. Run SETUP (if in suite)
3. Register MOCKs in mock registry
4. Execute test body:
   - When RUN SESSION encountered:
     - Check mock registry for match
     - If mocked: return mock value (no AI call)
     - If not mocked: execute normally (optional)
   - When ASSERT encountered:
     - Evaluate condition
     - If false: mark test FAILED, capture details
     - If true: continue
5. Run TEARDOWN (if in suite)
6. Report result (PASS/FAIL)
7. Cleanup test scope
```

### Mock Registry

```
MockRegistry = {
  "Researcher": {
    default: {data: "mocked research"},
    patterns: [
      {pattern: "research AI", returns: {topic: "AI"}},
      {pattern: "research quantum", returns: {topic: "quantum"}}
    ]
  },
  "Writer": {
    default: "Mocked article content",
    throws: null
  }
}
```

### Mock Matching

When session is about to run:

```
1. Look up agent name in MockRegistry
2. If agent has patterns:
   - Match task against each pattern
   - Return first matching mock
3. If no pattern match, use default mock
4. If mock has "throws", raise error
5. If no mock found, optionally run real session
```

---

## Examples

### Example 1: Basic Test Suite

```cheesecake
TEST SUITE "Calculator Tests":

  TEST "adds two numbers":
    VAR result = 2 + 2
    ASSERT result == 4
  END TEST

  TEST "handles negative numbers":
    VAR result = -5 + 3
    ASSERT result == -2
  END TEST

END TEST SUITE
```

### Example 2: Mocked Workflow Test

```cheesecake
TEST SUITE "Research Workflow Tests":

  SETUP:
    VAR researcher = NEW Researcher()
    VAR writer = NEW Writer()
  END SETUP

  TEST "produces article from research":
    MOCK Researcher RETURNS {
      findings: "Quantum computing uses qubits...",
      sources: ["arxiv.org", "nature.com"]
    }
    MOCK Writer RETURNS "# Quantum Computing\n\nAn introduction..."

    VAR findings = RUN SESSION(researcher): TASK: "Research quantum computing"
    VAR article = RUN SESSION(writer): TASK: "Write article" INPUT: {findings}

    ASSERT findings.sources.length == 2
    ASSERT article IS NOT NULL
    ASSERT **{article} is a well-structured article**
  END TEST

  TEST "handles research failure":
    MOCK Researcher THROWS "API rate limit exceeded"

    TRY:
      VAR findings = RUN SESSION(researcher): TASK: "Research topic"
      ASSERT FALSE MESSAGE "Should have thrown error"
    CATCH error:
      ASSERT error.message CONTAINS "rate limit"
    END TRY
  END TEST

END TEST SUITE
```

### Example 3: Task-Specific Mocks

```cheesecake
TEST "different responses for different tasks":
  MOCK Analyst FOR TASK MATCHING "*sentiment*" RETURNS {sentiment: "positive"}
  MOCK Analyst FOR TASK MATCHING "*summary*" RETURNS {summary: "Brief overview"}

  VAR sentiment = RUN SESSION(analyst): TASK: "Analyze sentiment of text"
  VAR summary = RUN SESSION(analyst): TASK: "Generate summary of document"

  ASSERT sentiment.sentiment == "positive"
  ASSERT summary.summary == "Brief overview"
END TEST
```

### Example 4: Semantic Assertions

```cheesecake
TEST "article meets quality standards":
  MOCK Writer RETURNS "# AI Trends\n\nArtificial intelligence continues..."

  VAR article = RUN SESSION(writer): TASK: "Write about AI"

  # Literal checks
  ASSERT article IS NOT NULL
  ASSERT article.length > 100

  # Semantic checks (AI evaluates)
  ASSERT **{article} has a clear introduction**
  ASSERT **{article} is written in professional tone**
  ASSERT **{article} does not contain placeholder text**
END TEST
```

---

## Test Output Format

### Summary View (Default)

```
Running tests in: workflow-tests.cheesecake

TEST SUITE: Research Workflow Tests
  ✓ produces article from research (0.1s)
  ✓ handles research failure (0.1s)
  ✗ validates input format (0.2s)
    FAILED: ASSERT input.format == "json"
    Expected: "json"
    Actual: "xml"
    at line 45

TEST SUITE: Writer Tests
  ✓ creates markdown output (0.1s)
  ✓ handles empty input (0.1s)

─────────────────────────────────────────
Tests: 5 total, 4 passed, 1 failed
Time: 0.6s
```

### Verbose View

```
Running tests in: workflow-tests.cheesecake

TEST SUITE: Research Workflow Tests
  SETUP: Loading fixtures...

  TEST: produces article from research
    MOCK: Researcher → {findings: "...", sources: [...]}
    MOCK: Writer → "# Quantum Computing..."
    RUN SESSION(researcher): Using mock response
    RUN SESSION(writer): Using mock response
    ASSERT findings.sources.length == 2: PASS
    ASSERT article IS NOT NULL: PASS
    ASSERT **article is well-structured**: PASS (AI: confidence 0.95)
    ✓ PASSED (0.1s)

  TEARDOWN: Cleanup complete
```

---

## Success Criteria

- [ ] TEST SUITE syntax defined and documented
- [ ] TEST syntax defined and documented
- [ ] MOCK syntax defined (simple, pattern, throws)
- [ ] ASSERT syntax defined (literal, semantic)
- [ ] VM test execution protocol documented
- [ ] /cheesecake test command created
- [ ] Test file covers all features
- [ ] CHANGELOG updated
- [ ] Backward compatible

---

## Estimated Line Counts

| File | Lines |
|------|-------|
| testing.md | 700+ |
| SKILL.md additions | 180+ |
| vm.md additions | 250+ |
| cheesecake-test.md | 200+ |
| test-testing-framework.cheesecake | 250+ |
| CHANGELOG additions | 80+ |
| MODULE-13-COMPLETE.md | 250+ |
| **Total** | **1,910+** |

---

## Implementation Order

1. Create testing.md (complete specification)
2. Update SKILL.md (add section 13, renumber)
3. Update vm.md (add section 11, renumber)
4. Create commands/cheesecake-test.md
5. Create test-testing-framework.cheesecake
6. Update CHANGELOG.md
7. Create MODULE-13-COMPLETE.md
8. Commit and push

---

Ready to begin implementation!
