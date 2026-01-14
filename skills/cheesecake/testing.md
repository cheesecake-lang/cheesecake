# CheeseCake Testing Framework Specification
# Purpose: Define testing constructs for workflow validation
# Part of: CheeseCake v0.0.2 - Module 13 (Testing Framework)
#
# This file specifies how testing works in CheeseCake.
# Use this to write tests for your workflows.
#
# Usage:
#   Referenced when executing /cheesecake test command
#
# Dependencies:
#   - SKILL.md (language specification)
#   - vm.md (execution semantics)
#
# Related:
#   - commands/cheesecake-test.md (test command)

---

# Testing Framework in CheeseCake

## 1. Overview

### Purpose

CheeseCake's testing framework enables **workflow validation** without incurring AI costs. Key capabilities:

1. **TEST SUITE**: Organize related tests with shared setup/teardown
2. **TEST**: Individual test cases with pass/fail reporting
3. **MOCK**: Replace agent sessions with fixed responses
4. **ASSERT**: Verify conditions (literal and semantic)

### Why Test Workflows?

| Problem | Solution |
|---------|----------|
| AI sessions are expensive | MOCKs avoid real API calls |
| AI outputs are non-deterministic | MOCKs return predictable data |
| Workflow logic needs validation | ASSERTs verify expected behavior |
| Tests need organization | TEST SUITE groups related tests |
| Quality needs verification | Semantic ASSERTs evaluate quality |

### Core Concepts

```
┌─────────────────────────────────────────────────────────────────┐
│                  CheeseCake Testing Framework                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  TEST SUITE "Name"                                              │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  SETUP:        (runs before each test)                    │  │
│  │    Load fixtures, initialize state                        │  │
│  │  END SETUP                                                │  │
│  │                                                           │  │
│  │  TEST "case 1":                                           │  │
│  │    MOCK Agent RETURNS value                               │  │
│  │    VAR result = RUN SESSION(agent): TASK: "..."           │  │
│  │    ASSERT result == expected                              │  │
│  │  END TEST                                                 │  │
│  │                                                           │  │
│  │  TEST "case 2":                                           │  │
│  │    ...                                                    │  │
│  │  END TEST                                                 │  │
│  │                                                           │  │
│  │  TEARDOWN:     (runs after each test)                     │  │
│  │    Cleanup resources                                      │  │
│  │  END TEARDOWN                                             │  │
│  └───────────────────────────────────────────────────────────┘  │
│  END TEST SUITE                                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. TEST SUITE Construct

### Purpose

`TEST SUITE` groups related tests with optional shared setup and teardown.

### Syntax

```cheesecake
TEST SUITE "suite name":
  [SETUP:
    # Statements to run before each test
  END SETUP]

  [TEARDOWN:
    # Statements to run after each test
  END TEARDOWN]

  TEST "test name":
    # Test body
  END TEST

  [... more TEST blocks ...]

END TEST SUITE
```

### Components

| Component | Required | Description |
|-----------|----------|-------------|
| `"suite name"` | Yes | Unique identifier for the suite |
| `SETUP` | No | Runs before each test in suite |
| `TEARDOWN` | No | Runs after each test in suite |
| `TEST` | Yes (at least 1) | Individual test cases |

### Rules

1. **Unique Names**: Suite name must be unique in file
2. **Order**: SETUP must come before tests, TEARDOWN after tests
3. **Execution**: Tests run in declaration order
4. **Isolation**: Each test has its own scope
5. **Failure Handling**: Failed test doesn't stop other tests

### Examples

#### Basic Suite

```cheesecake
TEST SUITE "Math Operations":

  TEST "addition works":
    ASSERT 2 + 2 == 4
  END TEST

  TEST "subtraction works":
    ASSERT 5 - 3 == 2
  END TEST

END TEST SUITE
```

#### Suite with Setup/Teardown

```cheesecake
TEST SUITE "Database Operations":

  SETUP:
    VAR db = MOCK_DATABASE()
    VAR test_user = {id: 1, name: "Test User"}
    db.insert(test_user)
  END SETUP

  TEARDOWN:
    db.clear()
    CLEANUP temp_files
  END TEARDOWN

  TEST "can retrieve user":
    VAR user = db.get(1)
    ASSERT user.name == "Test User"
  END TEST

  TEST "can update user":
    db.update(1, {name: "Updated Name"})
    VAR user = db.get(1)
    ASSERT user.name == "Updated Name"
  END TEST

END TEST SUITE
```

#### Multiple Suites in One File

```cheesecake
TEST SUITE "Input Validation":
  TEST "rejects empty input":
    # ...
  END TEST
END TEST SUITE

TEST SUITE "Output Formatting":
  TEST "formats JSON correctly":
    # ...
  END TEST
END TEST SUITE
```

---

## 3. TEST Construct

### Purpose

`TEST` defines an individual test case that either passes or fails.

### Syntax

```cheesecake
TEST "descriptive test name":
  # Test body: MOCKs, statements, ASSERTs
END TEST
```

### Rules

1. **Unique Name**: Test name must be unique within suite
2. **Isolation**: Variables don't leak between tests
3. **First Failure**: Test fails on first failed ASSERT
4. **Error Handling**: Uncaught errors also fail the test
5. **Return**: Tests don't return values

### Test Structure (AAA Pattern)

```cheesecake
TEST "descriptive name":
  # ARRANGE: Set up mocks and data
  MOCK Agent RETURNS expected_value
  VAR input = {data: "test"}

  # ACT: Execute the code being tested
  VAR result = RUN SESSION(agent): TASK: "Process" INPUT: {input}

  # ASSERT: Verify the outcome
  ASSERT result IS NOT NULL
  ASSERT result.status == "success"
END TEST
```

### Examples

#### Simple Test

```cheesecake
TEST "string concatenation":
  VAR greeting = "Hello, " + "World!"
  ASSERT greeting == "Hello, World!"
END TEST
```

#### Test with Mock

```cheesecake
TEST "researcher returns findings":
  MOCK Researcher RETURNS {
    topic: "AI",
    findings: ["Finding 1", "Finding 2"],
    confidence: 0.9
  }

  VAR researcher = NEW Researcher()
  VAR result = RUN SESSION(researcher): TASK: "Research AI trends"

  ASSERT result.topic == "AI"
  ASSERT result.findings.length == 2
  ASSERT result.confidence > 0.8
END TEST
```

#### Test with Error Handling

```cheesecake
TEST "handles API timeout gracefully":
  MOCK APIClient THROWS "Connection timeout"

  VAR client = NEW APIClient()

  TRY:
    VAR data = RUN SESSION(client): TASK: "Fetch data"
    ASSERT FALSE MESSAGE "Should have thrown"
  CATCH error:
    ASSERT error.message CONTAINS "timeout"
  END TRY
END TEST
```

---

## 4. MOCK Construct

### Purpose

`MOCK` replaces agent sessions with fixed responses, avoiding real AI calls.

### Syntax

```cheesecake
# Simple mock - all sessions return this value
MOCK AgentName RETURNS value

# Task-specific mock - only matches specific tasks
MOCK AgentName FOR TASK MATCHING "pattern" RETURNS value

# Error mock - simulates session failure
MOCK AgentName THROWS "error message"
```

### Mock Types

#### Simple Mock

Returns the same value for all sessions of that agent:

```cheesecake
MOCK Researcher RETURNS "Sample research data"

# All Researcher sessions now return "Sample research data"
VAR r1 = RUN SESSION(researcher): TASK: "Research A"  # → "Sample research data"
VAR r2 = RUN SESSION(researcher): TASK: "Research B"  # → "Sample research data"
```

#### Structured Mock

Returns structured data:

```cheesecake
MOCK Analyzer RETURNS {
  sentiment: "positive",
  confidence: 0.92,
  keywords: ["AI", "machine learning", "future"],
  summary: "The text expresses optimism about AI"
}
```

#### Task-Specific Mock

Returns different values based on task pattern:

```cheesecake
MOCK Writer FOR TASK MATCHING "*article*" RETURNS "Article content..."
MOCK Writer FOR TASK MATCHING "*summary*" RETURNS "Brief summary..."
MOCK Writer FOR TASK MATCHING "*tweet*" RETURNS "Short tweet text"

VAR article = RUN SESSION(writer): TASK: "Write an article"  # → "Article content..."
VAR summary = RUN SESSION(writer): TASK: "Write a summary"   # → "Brief summary..."
VAR tweet = RUN SESSION(writer): TASK: "Write a tweet"       # → "Short tweet text"
```

#### Error Mock

Simulates session failure:

```cheesecake
MOCK ExternalAPI THROWS "Rate limit exceeded"

TRY:
  VAR data = RUN SESSION(api): TASK: "Fetch"
CATCH error:
  # error.message == "Rate limit exceeded"
END TRY
```

### Pattern Matching

Task patterns use glob-style matching:

| Pattern | Matches |
|---------|---------|
| `"*"` | Any task |
| `"*article*"` | Tasks containing "article" |
| `"Write *"` | Tasks starting with "Write " |
| `"* report"` | Tasks ending with " report" |
| `"Analyze * data"` | "Analyze sales data", "Analyze user data" |

### Mock Scope

Mocks apply only within the current TEST block:

```cheesecake
TEST "test 1":
  MOCK Agent RETURNS "mock value"
  # Mock active here
END TEST

TEST "test 2":
  # Mock from test 1 NOT active here
  # Would need to define mock again
END TEST
```

### Mock Priority

When multiple mocks could match:

1. Task-specific patterns checked first (in declaration order)
2. Default mock used if no pattern matches
3. If no mock at all, behavior depends on test mode setting

```cheesecake
MOCK Agent FOR TASK MATCHING "specific task" RETURNS "specific"
MOCK Agent RETURNS "default"

RUN SESSION(agent): TASK: "specific task"  # → "specific"
RUN SESSION(agent): TASK: "other task"     # → "default"
```

### Examples

#### Complete Mock Example

```cheesecake
TEST "full workflow with mocks":
  # Mock all agents in the workflow
  MOCK Researcher RETURNS {
    data: "Research findings about quantum computing",
    sources: ["arxiv.org", "nature.com", "science.org"]
  }

  MOCK Analyst RETURNS {
    insights: ["Insight 1", "Insight 2"],
    recommendations: ["Rec 1", "Rec 2"]
  }

  MOCK Writer FOR TASK MATCHING "*blog*" RETURNS "# Blog Post\n\nContent..."
  MOCK Writer FOR TASK MATCHING "*report*" RETURNS "## Report\n\nFindings..."

  # Run workflow - all sessions use mocks
  VAR researcher = NEW Researcher()
  VAR analyst = NEW Analyst()
  VAR writer = NEW Writer()

  VAR research = RUN SESSION(researcher): TASK: "Research quantum"
  VAR analysis = RUN SESSION(analyst): TASK: "Analyze" INPUT: {research}
  VAR blog = RUN SESSION(writer): TASK: "Write blog post" INPUT: {analysis}
  VAR report = RUN SESSION(writer): TASK: "Write report" INPUT: {analysis}

  # Verify
  ASSERT research.sources.length == 3
  ASSERT analysis.insights.length == 2
  ASSERT blog CONTAINS "Blog Post"
  ASSERT report CONTAINS "Report"
END TEST
```

---

## 5. ASSERT Construct

### Purpose

`ASSERT` verifies that conditions are met, failing the test if not.

### Syntax

```cheesecake
# Basic assertion
ASSERT condition

# With custom message
ASSERT condition MESSAGE "custom failure message"

# Semantic assertion (AI-evaluated)
ASSERT **semantic condition**
```

### Literal Assertions

Direct value comparisons:

```cheesecake
# Equality
ASSERT value == expected
ASSERT value != unexpected

# Comparison
ASSERT value > minimum
ASSERT value >= minimum
ASSERT value < maximum
ASSERT value <= maximum

# Null checks
ASSERT value IS NULL
ASSERT value IS NOT NULL

# Boolean
ASSERT flag IS TRUE
ASSERT flag IS FALSE

# Contains
ASSERT list CONTAINS item
ASSERT string CONTAINS substring

# Pattern matching
ASSERT string MATCHES "pattern"

# Type checks
ASSERT value IS STRING
ASSERT value IS NUMBER
ASSERT value IS LIST
ASSERT value IS OBJECT
```

### Semantic Assertions

AI-evaluated quality checks:

```cheesecake
# Content quality
ASSERT **{article} is well-written and coherent**
ASSERT **{summary} captures the main points**
ASSERT **{response} answers the user's question**

# Structure
ASSERT **{document} has proper headings and sections**
ASSERT **{code} follows best practices**

# Negative checks
ASSERT **{text} does not contain offensive content**
ASSERT **{response} does not reveal sensitive information**

# Comparison
ASSERT **{output} is similar to {expected}**
ASSERT **{revised} is better than {original}**
```

### Custom Messages

Provide context on failure:

```cheesecake
ASSERT user.age >= 18 MESSAGE "User must be adult for this feature"
ASSERT response.status == 200 MESSAGE "API should return success status"
ASSERT **{article} is publication-ready** MESSAGE "Article needs more polish"
```

### Assertion Failure

When an assertion fails:

1. Test immediately stops
2. Failure details captured:
   - Assertion text
   - Expected vs actual (for literal)
   - AI evaluation (for semantic)
   - Line number
   - Custom message (if provided)
3. Test marked as FAILED
4. Continue to next test

### Examples

#### Comprehensive Assertions

```cheesecake
TEST "output validation":
  MOCK DataProcessor RETURNS {
    status: "success",
    records: 150,
    errors: [],
    output: "Processed data summary..."
  }

  VAR result = RUN SESSION(processor): TASK: "Process batch"

  # Status checks
  ASSERT result IS NOT NULL MESSAGE "Should return result"
  ASSERT result.status == "success" MESSAGE "Processing should succeed"

  # Numeric checks
  ASSERT result.records > 0 MESSAGE "Should process some records"
  ASSERT result.records == 150

  # Collection checks
  ASSERT result.errors IS NOT NULL
  ASSERT result.errors.length == 0 MESSAGE "Should have no errors"

  # String checks
  ASSERT result.output IS STRING
  ASSERT result.output.length > 10
  ASSERT result.output CONTAINS "summary"

  # Semantic checks
  ASSERT **{result.output} is a coherent summary**
  ASSERT **{result.output} does not contain placeholder text**
END TEST
```

---

## 6. Test Execution

### Test Discovery

The test runner discovers tests by:

1. Scanning specified file(s) for TEST SUITE and TEST blocks
2. Building test registry with metadata
3. Reporting discovered tests before execution

### Execution Order

```
For each TEST SUITE:
  For each TEST in suite:
    1. Create isolated test scope
    2. Run SETUP (if present)
    3. Initialize mock registry for this test
    4. Execute test body
    5. Run TEARDOWN (if present)
    6. Record result (PASS/FAIL)
    7. Cleanup test scope
  End For
End For
```

### Mock Resolution

When `RUN SESSION` is encountered during test:

```
1. Look up agent name in mock registry
2. If found:
   a. Check task-specific patterns (in order)
   b. If pattern matches, return that mock's value
   c. If no pattern matches, use default mock
   d. If mock has "throws", raise error instead
3. If not found:
   - In strict mode: fail test
   - In permissive mode: run real session (with warning)
4. Return mock value (no actual AI call)
```

### Result Recording

For each test:

```
TestResult = {
  suite: "Suite Name",
  name: "Test Name",
  status: PASS | FAIL | ERROR | SKIP,
  duration: 0.15s,
  failure: {
    assertion: "ASSERT x == y",
    expected: y,
    actual: x,
    line: 42,
    message: "Custom message"
  } | null,
  error: Error | null
}
```

---

## 7. Test Output

### Summary Format (Default)

```
Running tests: workflow-tests.cheesecake

TEST SUITE: Research Workflow
  ✓ returns valid research data (0.02s)
  ✓ handles empty topic (0.01s)
  ✗ validates source count (0.03s)
    FAILED: ASSERT sources.length >= 3
    Expected: >= 3
    Actual: 2
    at line 34

TEST SUITE: Writer Workflow
  ✓ produces markdown output (0.02s)
  ○ skipped: requires API key

────────────────────────────────────
Tests:  5 total
Passed: 3
Failed: 1
Skipped: 1
Time:   0.08s
```

### Verbose Format

```
Running tests: workflow-tests.cheesecake

TEST SUITE: Research Workflow
  SETUP: Initializing fixtures...
  SETUP: Complete

  TEST: returns valid research data
    MOCK Researcher → {data: "...", sources: [...]}
    EXECUTE: VAR researcher = NEW Researcher()
    EXECUTE: VAR result = RUN SESSION(researcher)...
      → Using mock (no AI call)
    ASSERT result IS NOT NULL: PASS
    ASSERT result.data.length > 0: PASS
    ASSERT **{result} is coherent**: PASS (AI confidence: 0.94)
    ✓ PASSED (0.02s)

  TEARDOWN: Cleanup complete
```

### Failure Details

```
✗ FAILED: validates source count

  Assertion: ASSERT sources.length >= 3
  Expected:  >= 3
  Actual:    2

  Test Code:
    31 |   VAR result = RUN SESSION(researcher): TASK: "Research"
    32 |   VAR sources = result.sources
    33 |
  > 34 |   ASSERT sources.length >= 3 MESSAGE "Need at least 3 sources"
    35 | END TEST

  Message: Need at least 3 sources
```

---

## 8. Best Practices

### 1. Descriptive Test Names

```cheesecake
# Good - describes what and expected outcome
TEST "researcher returns at least 3 sources for broad topics":
TEST "writer handles empty input by returning error message":

# Bad - vague
TEST "test1":
TEST "it works":
```

### 2. One Concept Per Test

```cheesecake
# Good - focused tests
TEST "validates email format":
  ASSERT is_valid_email("test@example.com") IS TRUE
END TEST

TEST "rejects invalid email":
  ASSERT is_valid_email("not-an-email") IS FALSE
END TEST

# Bad - testing multiple things
TEST "email validation":
  ASSERT is_valid_email("test@example.com") IS TRUE
  ASSERT is_valid_email("not-an-email") IS FALSE
  ASSERT is_valid_email("") IS FALSE
  # If first fails, others don't run
END TEST
```

### 3. Use Realistic Mock Data

```cheesecake
# Good - realistic data
MOCK Researcher RETURNS {
  topic: "Machine Learning",
  findings: "Recent advances in transformer architectures...",
  sources: ["arxiv.org/abs/2301.001", "nature.com/articles/s41586"],
  confidence: 0.89
}

# Bad - placeholder data
MOCK Researcher RETURNS {
  topic: "test",
  findings: "xxx",
  sources: ["a", "b"],
  confidence: 1.0
}
```

### 4. Test Edge Cases

```cheesecake
TEST SUITE "Edge Cases":

  TEST "handles empty input":
    MOCK Processor RETURNS {status: "error", message: "Empty input"}
    VAR result = RUN SESSION(processor): TASK: "Process" INPUT: {}
    ASSERT result.status == "error"
  END TEST

  TEST "handles very long input":
    VAR long_text = REPEAT("word ", 10000)
    MOCK Processor RETURNS {status: "success", truncated: true}
    VAR result = RUN SESSION(processor): TASK: "Process" INPUT: {text: long_text}
    ASSERT result.truncated IS TRUE
  END TEST

  TEST "handles special characters":
    MOCK Processor RETURNS {status: "success"}
    VAR result = RUN SESSION(processor): TASK: "Process <script>alert('xss')</script>"
    ASSERT result.status == "success"
  END TEST

END TEST SUITE
```

### 5. Use Semantic Assertions Wisely

```cheesecake
# Good - semantic for quality, literal for structure
TEST "article quality":
  MOCK Writer RETURNS article_content

  VAR article = RUN SESSION(writer): TASK: "Write"

  # Literal: verify structure
  ASSERT article IS NOT NULL
  ASSERT article.length > 500
  ASSERT article CONTAINS "# "  # Has headings

  # Semantic: verify quality
  ASSERT **{article} is coherent and well-organized**
  ASSERT **{article} has a clear conclusion**
END TEST

# Avoid - semantic for things that can be literal
TEST "bad example":
  ASSERT **{value} equals 42**  # Just use: ASSERT value == 42
END TEST
```

### 6. Organize with Suites

```cheesecake
# Good - logical grouping
TEST SUITE "Input Validation":
  TEST "rejects null":
  TEST "rejects empty string":
  TEST "accepts valid input":
END TEST SUITE

TEST SUITE "Output Formatting":
  TEST "formats as JSON":
  TEST "formats as XML":
  TEST "formats as CSV":
END TEST SUITE

# Bad - all in one suite
TEST SUITE "All Tests":
  TEST "input null":
  TEST "json output":
  TEST "xml output":
  TEST "input empty":
  # Hard to navigate
END TEST SUITE
```

### 7. Clean Up in Teardown

```cheesecake
TEST SUITE "File Operations":

  SETUP:
    VAR temp_dir = CREATE_TEMP_DIR()
  END SETUP

  TEARDOWN:
    DELETE temp_dir  # Always clean up
  END TEARDOWN

  TEST "creates output file":
    # Test uses temp_dir
  END TEST

END TEST SUITE
```

### 8. Document Test Intent

```cheesecake
# Tests for the research workflow's error handling
# These verify that failures are handled gracefully
TEST SUITE "Research Error Handling":

  # Verifies that network errors don't crash the workflow
  # and return appropriate error messages
  TEST "handles network timeout":
    MOCK Researcher THROWS "Network timeout after 30s"
    # ...
  END TEST

END TEST SUITE
```

---

## 9. Integration with Other Features

### With Events

Test event handlers:

```cheesecake
TEST "file_changed event triggers linting":
  MOCK Linter RETURNS {errors: [], warnings: []}

  # Emit test event
  EMIT file_changed(path: "src/app.ts", type: "modified")

  # Verify handler executed (check side effects)
  ASSERT lint_was_called IS TRUE
END TEST
```

### With INTERACTIVE

Test interactive workflows:

```cheesecake
TEST "interactive approval workflow":
  MOCK Reviewer RETURNS {quality: "high", approved: true}

  # In test mode, INTERACTIVE auto-selects first option
  # or can be configured to select specific option
  VAR result = RUN workflow_with_approval()

  ASSERT result.status == "approved"
END TEST
```

### With Cost Management

Mocks incur zero cost:

```cheesecake
CONFIG:
  BUDGET: $0.00  # Zero budget - only mocks should run
END CONFIG

TEST "workflow uses no real API calls":
  MOCK Researcher RETURNS "data"
  MOCK Writer RETURNS "content"

  VAR result = CALL research_and_write(topic: "test")

  # If any real session tried to run, budget would fail
  ASSERT result IS NOT NULL
END TEST
```

### With Checkpoints

Test checkpoint behavior:

```cheesecake
TEST "checkpoint saves state correctly":
  VAR state = {progress: 50, data: "test"}

  CHECKPOINT "test-checkpoint":
    SAVE: {state}
  END CHECKPOINT

  # Simulate resume
  RESTORE FROM "test-checkpoint"

  ASSERT state.progress == 50
  ASSERT state.data == "test"
END TEST
```

---

## 10. Examples

### Example 1: Complete Workflow Test

```cheesecake
# ============================================
# Tests for Research and Write Workflow
# ============================================

TEST SUITE "Research and Write Workflow":

  SETUP:
    VAR researcher = NEW Researcher()
    VAR writer = NEW Writer()
  END SETUP

  TEST "produces article from research":
    MOCK Researcher RETURNS {
      topic: "Quantum Computing",
      findings: "Quantum computers use qubits that can exist in superposition...",
      sources: ["arxiv.org", "nature.com", "ibm.com"]
    }

    MOCK Writer RETURNS "# Quantum Computing\n\nQuantum computing represents..."

    VAR research = RUN SESSION(researcher): TASK: "Research quantum computing"
    VAR article = RUN SESSION(writer):
      TASK: "Write article"
      INPUT: {research}

    # Structure checks
    ASSERT research.sources.length == 3
    ASSERT article CONTAINS "# Quantum"

    # Quality checks
    ASSERT **{article} is well-structured with introduction and body**
    ASSERT **{article} accurately reflects the research findings**
  END TEST

  TEST "handles empty research gracefully":
    MOCK Researcher RETURNS {
      topic: "Unknown Topic",
      findings: "",
      sources: []
    }

    MOCK Writer RETURNS "Unable to write article: insufficient research data"

    VAR research = RUN SESSION(researcher): TASK: "Research obscure topic"
    VAR article = RUN SESSION(writer):
      TASK: "Write article"
      INPUT: {research}

    ASSERT research.findings == ""
    ASSERT article CONTAINS "Unable to write"
  END TEST

  TEST "retries on research failure":
    MOCK Researcher THROWS "API temporarily unavailable"

    TRY:
      VAR research = RUN SESSION(researcher):
        TASK: "Research topic"
        RETRY: 3
    CATCH error:
      ASSERT error.message CONTAINS "unavailable"
    END TRY
  END TEST

END TEST SUITE
```

### Example 2: Testing Edge Cases

```cheesecake
TEST SUITE "Input Edge Cases":

  TEST "handles null input":
    MOCK Processor RETURNS {error: "Input cannot be null"}

    VAR result = RUN SESSION(processor): TASK: "Process" INPUT: NULL

    ASSERT result.error IS NOT NULL
    ASSERT result.error CONTAINS "null"
  END TEST

  TEST "handles very long text":
    VAR long_text = ""
    REPEAT 1000:
      long_text = long_text + "This is a test sentence. "
    END REPEAT

    MOCK Processor RETURNS {
      status: "success",
      truncated: true,
      processed_length: 50000
    }

    VAR result = RUN SESSION(processor):
      TASK: "Process"
      INPUT: {text: long_text}

    ASSERT result.status == "success"
    ASSERT result.truncated IS TRUE
  END TEST

  TEST "handles unicode correctly":
    VAR unicode_text = "Hello 世界 🌍 مرحبا"

    MOCK Processor RETURNS {
      status: "success",
      detected_languages: ["en", "zh", "ar"]
    }

    VAR result = RUN SESSION(processor):
      TASK: "Process"
      INPUT: {text: unicode_text}

    ASSERT result.detected_languages CONTAINS "zh"
    ASSERT result.detected_languages.length == 3
  END TEST

END TEST SUITE
```

### Example 3: Task-Specific Mocking

```cheesecake
TEST SUITE "Multi-Task Agent":

  TEST "responds differently to different tasks":
    # Different responses for different task types
    MOCK Assistant FOR TASK MATCHING "*summarize*" RETURNS {
      type: "summary",
      content: "Brief summary of the content..."
    }

    MOCK Assistant FOR TASK MATCHING "*translate*" RETURNS {
      type: "translation",
      content: "Translated text...",
      source_lang: "en",
      target_lang: "es"
    }

    MOCK Assistant FOR TASK MATCHING "*analyze*" RETURNS {
      type: "analysis",
      sentiment: "positive",
      topics: ["technology", "innovation"]
    }

    VAR assistant = NEW Assistant()

    VAR summary = RUN SESSION(assistant): TASK: "Please summarize this document"
    VAR translation = RUN SESSION(assistant): TASK: "Translate to Spanish"
    VAR analysis = RUN SESSION(assistant): TASK: "Analyze the sentiment"

    ASSERT summary.type == "summary"
    ASSERT translation.type == "translation"
    ASSERT translation.target_lang == "es"
    ASSERT analysis.type == "analysis"
    ASSERT analysis.sentiment == "positive"
  END TEST

END TEST SUITE
```

---

## 11. Quick Reference

### TEST SUITE

```cheesecake
TEST SUITE "name":
  [SETUP: ... END SETUP]
  [TEARDOWN: ... END TEARDOWN]
  TEST "name": ... END TEST
END TEST SUITE
```

### TEST

```cheesecake
TEST "descriptive name":
  # Arrange, Act, Assert
END TEST
```

### MOCK

```cheesecake
MOCK Agent RETURNS value
MOCK Agent FOR TASK MATCHING "pattern" RETURNS value
MOCK Agent THROWS "error"
```

### ASSERT

```cheesecake
ASSERT condition
ASSERT condition MESSAGE "custom message"
ASSERT **semantic condition**
```

### Assertion Operators

| Operator | Example |
|----------|---------|
| `==` | `ASSERT x == 5` |
| `!=` | `ASSERT x != 0` |
| `>`, `<`, `>=`, `<=` | `ASSERT x > 0` |
| `IS NULL` | `ASSERT x IS NULL` |
| `IS NOT NULL` | `ASSERT x IS NOT NULL` |
| `IS TRUE/FALSE` | `ASSERT flag IS TRUE` |
| `CONTAINS` | `ASSERT list CONTAINS item` |
| `MATCHES` | `ASSERT str MATCHES "pattern"` |

---

## 12. Glossary

| Term | Definition |
|------|------------|
| **TEST SUITE** | Container for related tests with shared setup |
| **TEST** | Individual test case that passes or fails |
| **MOCK** | Fake agent response for testing without AI |
| **ASSERT** | Verification that a condition is true |
| **SETUP** | Code that runs before each test |
| **TEARDOWN** | Code that runs after each test |
| **Semantic Assertion** | AI-evaluated quality check |
| **Test Runner** | System that discovers and executes tests |

---

**Module 13: Testing Framework - Complete Specification**
