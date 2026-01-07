# CheeseCake Language Specification
# Purpose: Complete language definition for AI agents to understand and execute .cheesecake files
# Part of: CheeseCake v0.0.1 - Module 2 (Language Specification)
# Updated: v0.0.2 - Added PHASE construct for progress tracking
#
# This file teaches AI agents the complete CheeseCake syntax and semantics.
# When you read a .cheesecake file, use this specification to understand and execute it.
#
# Usage:
#   This skill is automatically loaded when executing .cheesecake files
#
# Dependencies:
#   - vm.md (execution semantics)
#   - Claude Code Task tool (for creating sub-agent sessions)
#
# Related:
#   - syntax-reference.md (quick reference)
#   - helper.md (for generating .cheesecake files)

---
name: cheesecake
description: CheeseCake language interpreter. Use when reading or executing .cheesecake files.
---

# CheeseCake Language Specification v0.0.2

## Overview

**CheeseCake** is a structured, object-oriented programming language for orchestrating AI agents. It treats long-running AI sessions as Turing-complete computers and provides explicit keywords for unambiguous control flow.

### Core Philosophy

1. **AI-Native**: Designed for AI interpretation, not machine parsing
2. **Structured**: Explicit keywords (`AGENT`, `VAR`, `RUN`, `SESSION`) for clarity
3. **Object-Oriented**: First-class support for classes, inheritance, composition
4. **Portable**: Works across AI agents (abstraction layer)
5. **Resumable**: Built-in checkpoints for long-running workflows

### Execution Model

- **Session-as-Runtime**: The AI session executing the .cheesecake file IS the interpreter
- **Sub-agent Spawning**: Use Claude Code's Task tool to create sessions for each agent
- **Semantic Evaluation**: Conditions in `**...**` are evaluated by AI understanding, not regex
- **State Persistence**: Checkpoints save to `.cheesecake/state/` directory

---

## 1. File Structure

### Basic Structure

```cheesecake
# Comments start with # and go to end of line

# Imports (optional, at top)
IMPORT "module.cheesecake" AS alias

# Skill definitions (optional)
SKILL skill_name:
  CAPABILITIES: [...]
  REQUIRES: [...]

# Agent definitions (optional)
AGENT AgentName:
  MODEL: model_name
  SKILLS: [...]
  PROMPT: "..."

# Main execution logic
VAR agent = NEW AgentName()
VAR result = RUN SESSION(agent): TASK: "Do something"
PRINT result
```

### File Conventions

- **File extension**: `.cheesecake`
- **Encoding**: UTF-8
- **Indentation**: 2 spaces (consistency matters for readability)
- **Case sensitivity**: Keywords are UPPERCASE, user-defined names are flexible
- **Line breaks**: Use newlines for readability, but not required after every statement

---

## 2. Comments

```cheesecake
# Single-line comment

# Multi-line comments use multiple # lines
# Like this
# And this

# Section headers for organization
# ============================================
# SECTION: Agent Definitions
# Purpose: Define all agents used in this workflow
# ============================================
```

**Purpose**: Documentation, explaining non-obvious logic

---

## 3. SKILL Definitions (Traits/Interfaces)

Skills are reusable capability bundles that can be attached to agents.

### Basic Syntax

```cheesecake
SKILL skill_name:
  CAPABILITIES:
    - capability 1
    - capability 2
    - capability 3
  REQUIRES:
    - requirement 1
    - requirement 2
```

### Inheritance

```cheesecake
SKILL advanced_skill EXTENDS base_skill:
  CAPABILITIES:
    - additional capability 1
    - additional capability 2
```

**When inheriting**: The new skill gets all capabilities from the parent, plus its own.

### Examples

```cheesecake
# Basic research skill
SKILL web-research:
  CAPABILITIES:
    - search the web
    - fetch URLs
    - parse HTML
    - extract information
  REQUIRES:
    - internet access

# Extending with academic capabilities
SKILL academic-research EXTENDS web-research:
  CAPABILITIES:
    - access academic databases
    - parse academic papers
    - extract citations
  REQUIRES:
    - academic API access
```

### Semantic Note

**CAPABILITIES** and **REQUIRES** are natural language descriptions. The AI interprets what they mean. They're NOT executed as code - they inform the AI about what an agent can do when this skill is attached.

---

## 4. AGENT Definitions (Classes)

Agents are templates (like classes) for creating sessions. Each agent has a model, skills, and a system prompt.

### Basic Syntax

```cheesecake
AGENT AgentName:
  MODEL: model_name    # sonnet, opus, haiku
  SKILLS: [skill1, skill2]  # Array of skills
  PROMPT: "System prompt for this agent"
```

### Inheritance

```cheesecake
AGENT ChildAgent EXTENDS ParentAgent:
  MODEL: opus          # Override parent's model
  SKILLS: [+new_skill] # Add to parent's skills (+ means append)
  PROMPT: PARENT + "Additional instructions"  # Extend parent's prompt
```

### Composition (Implements Multiple Skills)

```cheesecake
AGENT FullStackDev IMPLEMENTS [frontend, backend, devops]:
  MODEL: sonnet
  PROMPT: "You are a full-stack developer."
```

### Examples

```cheesecake
# Basic agent
AGENT Researcher:
  MODEL: sonnet  # Using Sonnet for cost-efficiency
  SKILLS: [web-research, data-analysis]
  PROMPT: "You are a thorough researcher who finds accurate, well-sourced information."

# Inheritance example
AGENT SeniorResearcher EXTENDS Researcher:
  MODEL: opus  # Senior researcher uses more powerful model
  SKILLS: [+strategic-thinking, +synthesis]  # Adds to parent's skills
  PROMPT: PARENT + "You also provide strategic recommendations based on your research."

# Composition example
AGENT DataScientist IMPLEMENTS [data-analysis, ml-modeling, visualization]:
  MODEL: sonnet
  PROMPT: "You are a data scientist skilled in analysis, ML, and visualization."
```

### Model Options

- `sonnet` - Claude Sonnet (balanced, cost-effective)
- `opus` - Claude Opus (most powerful)
- `haiku` - Claude Haiku (fastest, cheapest)

---

## 5. Variables & Assignment

### Variable Declaration

```cheesecake
VAR variable_name = value     # Mutable variable
CONST constant_name = value   # Immutable variable
```

### Valid Values

```cheesecake
# NULL (uninitialized)
VAR result = NULL

# Agent instances
VAR researcher = NEW Researcher()

# Session outputs (strings, objects, arrays)
VAR findings = RUN SESSION(researcher): TASK: "Research X"

# Loaded data
VAR config = LOAD "config.json"

# Numbers, strings, booleans
VAR count = 5
VAR name = "CheeseCake"
VAR enabled = TRUE
```

### Assignment Rules

- `VAR` can be reassigned: `result = new_value`
- `CONST` cannot be reassigned (immutable)
- Variables must be declared before use
- Variable names are case-sensitive

### Examples

```cheesecake
# Declare and initialize
VAR researcher = NEW Researcher()
VAR findings = NULL  # Will be assigned later

# Run a session and store result
findings = RUN SESSION(researcher): TASK: "Research quantum computing"

# Constants for configuration
CONST max_retries = 5
CONST timeout = 30s

# Cannot reassign const (would error)
# max_retries = 10  # ERROR!
```

---

## 6. SESSION & RUN

### Creating a Session

A **SESSION** is an instance of an agent, configured with a specific task.

```cheesecake
# Create a session object (not yet executed)
VAR session_obj = SESSION(agent_instance):
  TASK: "Natural language task description"
  INPUT: {var1, var2}  # Optional: input data
  CONTEXT: {key: value}  # Optional: additional context
  TIMEOUT: 30s  # Optional: timeout
  RETRY: 3  # Optional: retry count
```

### Running a Session

**RUN** executes a session and returns the output.

```cheesecake
# Execute the session
VAR result = RUN session_obj

# Inline: create and run in one statement
VAR result = RUN SESSION(agent): TASK: "Do something"
```

### Full Example

```cheesecake
# Create agent
VAR researcher = NEW Researcher()

# Create session
VAR research_session = SESSION(researcher):
  TASK: "Find recent breakthroughs in quantum computing"
  CONTEXT: {
    domain: "physics",
    depth: "comprehensive",
    sources: "peer-reviewed only"
  }
  TIMEOUT: 60s
  RETRY: 2

# Run the session
VAR findings = RUN research_session

# Or do it inline
VAR summary = RUN SESSION(researcher):
  TASK: "Summarize these findings into 500 words"
  INPUT: {findings}
```

### TASK Syntax

The `TASK` field is a natural language instruction. You can use variable interpolation:

```cheesecake
VAR topic = "quantum computing"
VAR result = RUN SESSION(agent):
  TASK: "Research {topic} and provide a summary"  # {topic} gets replaced
  INPUT: {data: previous_results}
```

### INPUT vs CONTEXT

- **INPUT**: Direct data to process (like function arguments)
- **CONTEXT**: Additional configuration/hints for how to process

---

## 7. Control Flow

### Sequential Execution (Explicit)

```cheesecake
SEQUENCE:
  VAR step1 = RUN SESSION(agent1): TASK: "Do first thing"
  VAR step2 = RUN SESSION(agent2): TASK: "Do second thing" INPUT: {step1}
  VAR step3 = RUN SESSION(agent3): TASK: "Do third thing" INPUT: {step2}
END SEQUENCE
```

**Default**: Statements execute sequentially unless in a `PARALLEL` block.

### Parallel Execution

```cheesecake
PARALLEL:
  VAR result1 = RUN SESSION(agent1): TASK: "Task 1"
  VAR result2 = RUN SESSION(agent2): TASK: "Task 2"
  VAR result3 = RUN SESSION(agent3): TASK: "Task 3"
END PARALLEL

# All three sessions run concurrently
# Execution continues after all complete
```

### Parallel with Join Strategies

```cheesecake
# Wait for first to complete (race)
PARALLEL JOIN(first):
  VAR fast = RUN SESSION(api1): TASK: "Fetch from fast source"
  VAR slow = RUN SESSION(api2): TASK: "Fetch from slow source"
END PARALLEL
# Only one completes, winner's result is used

# Wait for any N
PARALLEL JOIN(any, 2):  # Wait for any 2 out of 3
  VAR r1 = RUN SESSION(agent): TASK: "Option 1"
  VAR r2 = RUN SESSION(agent): TASK: "Option 2"
  VAR r3 = RUN SESSION(agent): TASK: "Option 3"
END PARALLEL
```

### Conditionals

```cheesecake
IF **{variable} meets some semantic condition**:
  # Statements if true
ELIF **{variable} meets different condition**:
  # Statements if this is true
ELSE:
  # Statements if all false
END IF
```

**Semantic Conditions**: Text in `**...**` is evaluated by AI understanding, not regex or boolean logic. Examples:

```cheesecake
IF **{findings} shows significant progress in the field**:
  LOG "Exciting developments!"
END IF

IF **{feedback} indicates major structural issues**:
  # Do major revision
ELSE:
  # Do minor polish
END IF
```

### Choice (AI Selects Option)

```cheesecake
CHOICE ON **criteria for selection**:
  OPTION "label1":
    # Statements for option 1
  OPTION "label2":
    # Statements for option 2
  OPTION "label3":
    # Statements for option 3
END CHOICE
```

**How it works**: The AI evaluates the criteria and selects the most appropriate option.

```cheesecake
CHOICE ON **project complexity level (simple, moderate, complex)**:
  OPTION "simple":
    VAR impl = RUN SESSION(junior_dev): TASK: "Implement feature"
  OPTION "moderate":
    VAR impl = RUN SESSION(senior_dev): TASK: "Implement feature"
  OPTION "complex":
    VAR design = RUN SESSION(architect): TASK: "Design solution first"
    VAR impl = RUN SESSION(senior_dev): TASK: "Implement" INPUT: {design}
END CHOICE
```

### Phase Blocks (v0.0.2+)

**Purpose**: Organize workflow into logical phases for better progress tracking and visualization.

```cheesecake
PHASE "phase-name":
  # Statements for this phase
  # Can include any valid CheeseCake code
END PHASE
```

**Key Points:**
- **Optional**: Phases are purely organizational, not functional
- **Sequential**: Phases execute one after another
- **Progress tracking**: VM shows which phase is currently running
- **Any code**: Can contain PARALLEL blocks, loops, conditions, etc.
- **Unique names**: Phase names must be unique within a workflow
- **Resumability**: Checkpoints between phases enable resume after failure

**Example:**

```cheesecake
# Multi-phase research workflow
# Phases provide clear progress visualization

PHASE "Research":
  # Research phase with parallel data gathering
  VAR researcher = NEW Researcher()

  PARALLEL:
    VAR academic = RUN SESSION(researcher):
      TASK: "Find academic papers on quantum computing"

    VAR industry = RUN SESSION(researcher):
      TASK: "Analyze industry developments"

    VAR market = RUN SESSION(researcher):
      TASK: "Evaluate market trends"
  END PARALLEL

  # Save checkpoint after research
  CHECKPOINT "research-complete":
    SAVE: {academic, industry, market}
  END CHECKPOINT
END PHASE

PHASE "Analysis":
  # Synthesis phase
  VAR analyst = NEW Analyst()

  VAR insights = RUN SESSION(analyst):
    TASK: "Synthesize research findings into key insights"
    INPUT: {academic, industry, market}

  CHECKPOINT "analysis-complete":
    SAVE: {insights}
  END CHECKPOINT
END PHASE

PHASE "Writing":
  # Iterative writing phase
  VAR writer = NEW Writer()
  VAR editor = NEW Editor()

  VAR draft = RUN SESSION(writer):
    TASK: "Write article based on insights"
    INPUT: {insights}

  # Iterative refinement
  LOOP UNTIL **{draft} meets publication standards** MAX 5:
    VAR feedback = RUN SESSION(editor):
      TASK: "Review and provide feedback"
      INPUT: {draft}

    VAR draft = RUN SESSION(writer):
      TASK: "Revise based on feedback"
      INPUT: {draft, feedback}
  END LOOP

  CHECKPOINT "writing-complete":
    SAVE: {draft}
  END CHECKPOINT
END PHASE

PHASE "Output":
  # Final output phase
  SAVE draft TO "output/article.md"
  LOG SUCCESS: "Article complete!"
END PHASE
```

**Progress Visualization:**

When executing a workflow with phases, the VM displays:

```
╔═══════════════════════════════════════════════════════════╗
║  Executing: research-workflow.cheesecake                  ║
╚═══════════════════════════════════════════════════════════╝

[■■■■■■□□□□] 60% complete

✓ Phase 1: Research           [DONE]     8.5s
✓ Phase 2: Analysis           [DONE]     4.2s
→ Phase 3: Writing            [RUNNING]  2.1s
○ Phase 4: Output             [PENDING]

Tokens: 8,420 used | ~4,000 remaining
Time: 14.8s elapsed | ~7s remaining
```

**When to Use PHASE Blocks:**

1. **Long workflows** (>5 minutes execution time)
2. **Multiple distinct stages** with different agents or tasks
3. **When checkpoints are needed** between major stages
4. **Complex workflows** where progress visibility helps user confidence
5. **Debugging** - easier to identify which phase failed

**When NOT to Use PHASE Blocks:**

1. **Simple workflows** (1-3 operations)
2. **Already clear structure** - don't over-organize
3. **Single agent, single task** - unnecessary wrapper

**Rules:**

- Phase names must be **unique strings**
- Phases execute **sequentially** (not parallel)
- **Nested phases not allowed** (no PHASE inside PHASE)
- **Works with all constructs** (can contain loops, conditionals, parallel blocks)
- **No return values** - phases are organizational, not functional

**Integration with Checkpoints:**

Phases work well with checkpoints for resumability:

```cheesecake
PHASE "Expensive Research":
  # If this phase fails, can resume without re-doing
  VAR data = RUN SESSION(expensive_agent): TASK: "..."

  CHECKPOINT "research-done":
    SAVE: {data}
  END CHECKPOINT
END PHASE

PHASE "Analysis":
  # If we resume, we start here with loaded data
  VAR result = RUN SESSION(analyst): TASK: "..." INPUT: {data}
END PHASE
```

**Cost Estimation:**

When using `/cheesecake estimate` or `--dry-run`, phases provide:
- **Per-phase cost breakdown**
- **Progress prediction** (which phase will take longest)
- **Optimization suggestions** (identify expensive phases)

Example output:
```
Phase 1: Research        $0.06  (12%)   8s
Phase 2: Analysis        $0.04  (8%)    4s
Phase 3: Writing         $0.35  (70%)   25s   ← Most expensive
Phase 4: Output          $0.00  (0%)    0.5s
```

**Note**: Phases are **optional in v0.0.1**. They're added in v0.0.2 to enhance progress tracking, but existing workflows without phases continue to work perfectly.

---

## 8. Loops

### Fixed Iteration (REPEAT)

```cheesecake
REPEAT count:
  # Statements (executed count times)
END REPEAT

# With index variable
REPEAT count AS index:
  # index goes from 0 to count-1
  PRINT "Iteration {index}"
END REPEAT
```

### For-Each Loop

```cheesecake
FOR item IN collection:
  # Process each item
  VAR result = RUN SESSION(agent): TASK: "Process {item}"
END FOR

# With index
FOR item, index IN collection:
  PRINT "Processing item {index}: {item}"
END FOR
```

### Parallel For-Each

```cheesecake
PARALLEL FOR item IN items:
  VAR processed = RUN SESSION(processor): TASK: "Process {item}"
END PARALLEL FOR

# All items processed concurrently
```

### Unbounded Loops (Semantic Exit)

```cheesecake
# Loop until condition is met
LOOP UNTIL **{variable} meets exit condition** MAX max_iterations:
  # Statements
  # Update variable
END LOOP

# Loop while condition is true
WHILE **{variable} meets continue condition** MAX max_iterations:
  # Statements
END WHILE

# Infinite loop (requires MAX for safety)
LOOP MAX 100:
  # Statements
  # Must have some way to break or MAX will stop it
END LOOP
```

### Loop Safety

**IMPORTANT**: All unbounded loops (`LOOP UNTIL`, `WHILE`, `LOOP`) MUST have a `MAX` limit to prevent infinite execution.

### Examples

```cheesecake
# Fixed iteration
REPEAT 3:
  VAR draft = RUN SESSION(writer):
    TASK: "Improve the draft"
    INPUT: {draft}
END REPEAT

# For-each
FOR task IN todo_list:
  VAR result = RUN SESSION(worker): TASK: "Complete {task}"
  LOG "Completed: {task}"
END FOR

# Semantic loop (iterative refinement)
VAR draft = RUN SESSION(writer): TASK: "Write initial draft"

LOOP UNTIL **{draft} meets publication standards: well-structured, accurate, engaging** MAX 5:
  VAR feedback = RUN SESSION(editor): TASK: "Review {draft}"

  IF **{feedback} indicates major issues**:
    VAR draft = RUN SESSION(writer): TASK: "Major revision" INPUT: {draft, feedback}
  ELSE:
    VAR draft = RUN SESSION(writer): TASK: "Minor polish" INPUT: {draft, feedback}
  END IF

  LOG "Completed revision iteration"
END LOOP
```

---

## 9. State & Persistence

### Checkpoints

Checkpoints save workflow state to disk for resumability.

```cheesecake
# Save checkpoint
CHECKPOINT "checkpoint_name":
  SAVE: {var1, var2, var3}
  TO: ".cheesecake/state/"  # Optional: custom location
END CHECKPOINT

# Check if checkpoint exists
IF CHECKPOINT_EXISTS("checkpoint_name"):
  RESTORE FROM "checkpoint_name"
  LOG "Resumed from checkpoint"
ELSE:
  # Execute from scratch
  VAR var1 = ...
END IF
```

### Memory (Cross-Session Persistence)

Memory blocks persist data across multiple executions.

```cheesecake
# Define memory
MEMORY memory_name:
  key1: "value1"
  key2: [list, of, values]
  key3: {nested: "object"}
END MEMORY

# Update memory
MEMORY memory_name.key1 = "new value"
MEMORY memory_name.key3.nested = "updated"

# Append to list
MEMORY memory_name.key2.APPEND("new item")

# Load memory
VAR data = LOAD MEMORY memory_name
```

### Examples

```cheesecake
# Checkpoint for long workflow
PARALLEL:
  VAR academic = RUN SESSION(researcher): TASK: "Academic research"
  VAR industry = RUN SESSION(researcher): TASK: "Industry research"
  VAR market = RUN SESSION(analyst): TASK: "Market analysis"
END PARALLEL

# Save checkpoint after expensive operation
CHECKPOINT "research-complete":
  SAVE: {academic, industry, market}
END CHECKPOINT

# Resume from checkpoint if exists
IF CHECKPOINT_EXISTS("outline-complete"):
  RESTORE FROM "outline-complete"
ELSE:
  VAR combined = RUN SESSION(analyst): TASK: "Synthesize" INPUT: {academic, industry, market}
  VAR outline = RUN SESSION(writer): TASK: "Create outline" INPUT: {combined}

  CHECKPOINT "outline-complete":
    SAVE: {combined, outline}
  END CHECKPOINT
END IF

# Memory for project state
MEMORY project_state:
  phase: "research"
  history: []
END MEMORY

# Update history
MEMORY project_state.history.APPEND({
  timestamp: NOW(),
  action: "Completed research phase",
  details: "Found {academic.length} papers"
})
```

---

## 10. Error Handling

### Try/Catch/Finally

```cheesecake
TRY:
  # Statements that might fail
  VAR result = RUN SESSION(agent): TASK: "Risky operation"
CATCH error:
  # Handle error
  LOG ERROR: "Failed: {error.message}"
  VAR result = fallback_value
FINALLY:
  # Always executed (cleanup)
  CLEANUP temporary_files
END TRY
```

### Retry with Backoff

```cheesecake
VAR data = RUN SESSION(api_agent):
  TASK: "Fetch from external API"
  RETRY: 3  # Try up to 3 times
  BACKOFF: exponential  # Wait longer between retries
  ON_FAIL: THROW "Could not fetch data after 3 attempts"
```

### Examples

```cheesecake
# Graceful fallback
TRY:
  VAR live_data = RUN SESSION(api_agent): TASK: "Fetch live data"
CATCH error:
  LOG WARNING: "API unavailable, using cache"
  VAR live_data = RUN SESSION(cache): TASK: "Get cached data"
END TRY

# Cleanup resources
VAR temp_file = NULL
TRY:
  temp_file = CREATE_TEMP_FILE()
  VAR result = RUN SESSION(processor): TASK: "Process file" INPUT: {temp_file}
FINALLY:
  IF temp_file IS NOT NULL:
    DELETE temp_file
  END IF
END TRY
```

---

## 11. Functions

Functions are reusable workflows with parameters and return values.

### Syntax

```cheesecake
FUNCTION function_name(param1, param2, ...):
  # Statements
  VAR result = ...
  RETURN result
END FUNCTION

# Call function
VAR output = CALL function_name(arg1, arg2, ...)
```

### Examples

```cheesecake
# Research workflow function
FUNCTION research_topic(topic, depth):
  VAR researcher = NEW Researcher()

  VAR findings = RUN SESSION(researcher):
    TASK: "Research {topic} at {depth} depth"

  VAR summary = RUN SESSION(researcher):
    TASK: "Summarize {findings} in bullet points"

  RETURN {findings, summary}
END FUNCTION

# Usage
VAR quantum_research = CALL research_topic(topic: "quantum computing", depth: "comprehensive")
PRINT quantum_research.summary

# Multi-step function
FUNCTION analyze_and_report(data, format):
  VAR analyst = NEW Analyst()
  VAR writer = NEW Writer()

  VAR analysis = RUN SESSION(analyst): TASK: "Analyze {data}"
  VAR report = RUN SESSION(writer):
    TASK: "Create {format} report from {analysis}"

  RETURN report
END FUNCTION
```

---

## 12. Modules & Imports

### Import Syntax

```cheesecake
# Import entire module
IMPORT "path/to/module.cheesecake" AS alias

# Use imported definitions
VAR agent = NEW alias.AgentName()
VAR result = CALL alias.function_name(args)
```

### Export Syntax

```cheesecake
# In the module file
EXPORT AGENT MyAgent
EXPORT SKILL MySkill
EXPORT FUNCTION my_function
```

### Examples

```cheesecake
# common/agents.cheesecake
AGENT Researcher:
  MODEL: sonnet
  SKILLS: [web-research]

AGENT Writer:
  MODEL: opus
  SKILLS: [content-creation]

EXPORT AGENT Researcher
EXPORT AGENT Writer

# main.cheesecake
IMPORT "common/agents.cheesecake" AS agents

VAR researcher = NEW agents.Researcher()
VAR writer = NEW agents.Writer()

VAR findings = RUN SESSION(researcher): TASK: "Research AI trends"
VAR article = RUN SESSION(writer): TASK: "Write article" INPUT: {findings}
```

---

## 13. Built-in Functions & Commands

### Logging

```cheesecake
LOG "message"                    # Info level
LOG INFO: "message"              # Explicit info
LOG WARNING: "message"           # Warning
LOG ERROR: "message"             # Error
LOG SUCCESS: "message"           # Success
```

### Output

```cheesecake
PRINT value                      # Print to output
SAVE value TO "path/to/file"    # Save to file
```

### Data Operations

```cheesecake
LOAD "path/to/file"              # Load file contents
FIRST(list)                      # Get first item
LAST(list)                       # Get last item
LENGTH(list)                     # Get length
REMOVE(list, item)               # Remove item from list
```

### Time

```cheesecake
NOW()                            # Current timestamp
```

### Utility

```cheesecake
CLEANUP resource                 # Cleanup/delete resource
CREATE_TEMP_FILE()              # Create temporary file
DELETE file                      # Delete file
```

---

## 14. Operators & Comparisons

### Assignment

```cheesecake
=                                # Assign value
```

### Comparison (in semantic conditions)

Within `**...**` blocks, use natural language:

```cheesecake
IF **{value} is greater than 100**:
IF **{status} equals "complete"**:
IF **{list} contains item "X"**:
IF **{draft} is ready for publication**:
```

### Logical (in semantic conditions)

```cheesecake
IF **{a} is true AND {b} is false**:
IF **{x} is valid OR {y} is available**:
IF **NOT {failed}**:
```

---

## 15. Best Practices

### 1. Use Descriptive Names

```cheesecake
# Good
VAR research_findings = RUN SESSION(researcher): TASK: "Research quantum"

# Bad
VAR x = RUN SESSION(r): TASK: "Research quantum"
```

### 2. Add Comments for Complex Logic

```cheesecake
# This loop refines the draft until it meets publication standards
# We limit to MAX 5 iterations to prevent infinite loops
# The semantic condition is evaluated by AI understanding
LOOP UNTIL **{draft} is publication-ready** MAX 5:
  VAR feedback = RUN SESSION(editor): TASK: "Review {draft}"
  VAR draft = RUN SESSION(writer): TASK: "Revise" INPUT: {draft, feedback}
END LOOP
```

### 3. Use Checkpoints for Long Workflows

```cheesecake
# After expensive parallel research
CHECKPOINT "research-complete":
  SAVE: {findings}
END CHECKPOINT
```

### 4. Handle Errors Gracefully

```cheesecake
TRY:
  VAR result = RUN SESSION(api): TASK: "Fetch data"
CATCH error:
  VAR result = fallback_data
  LOG WARNING: "Using fallback due to: {error}"
END TRY
```

### 5. Use Functions for Reusability

```cheesecake
# Define once, use many times
FUNCTION process_document(doc):
  VAR processed = RUN SESSION(processor): TASK: "Process {doc}"
  RETURN processed
END FUNCTION

# Use
FOR doc IN documents:
  VAR result = CALL process_document(doc)
END FOR
```

---

## 16. Complete Example

Here's a comprehensive example using many CheeseCake features:

```cheesecake
# ============================================
# COMPLETE RESEARCH & WRITE WORKFLOW
# Demonstrates: agents, parallel, loops, checkpoints, functions
# ============================================

# ============================================
# SECTION: Skill & Agent Definitions
# ============================================

SKILL research-capabilities:
  CAPABILITIES:
    - web search
    - data analysis
  REQUIRES:
    - internet access

AGENT Researcher:
  MODEL: sonnet
  SKILLS: [research-capabilities]
  PROMPT: "You are a thorough researcher."

AGENT Writer:
  MODEL: opus
  SKILLS: [content-creation]
  PROMPT: "You are a creative writer."

AGENT Editor EXTENDS Writer:
  PROMPT: PARENT + "You are also a meticulous editor."

# ============================================
# SECTION: Helper Functions
# ============================================

FUNCTION research_and_synthesize(topic, sources):
  VAR researcher = NEW Researcher()

  PARALLEL FOR source IN sources:
    VAR data = RUN SESSION(researcher):
      TASK: "Research {topic} from {source}"
  END PARALLEL FOR

  VAR synthesis = RUN SESSION(researcher):
    TASK: "Synthesize all research data"

  RETURN synthesis
END FUNCTION

# ============================================
# SECTION: Main Workflow
# ============================================

# Configuration
CONST topic = "quantum computing advances"
CONST sources = ["academic papers", "industry news", "market reports"]
CONST max_revisions = 5

# Create agent instances
VAR researcher = NEW Researcher()
VAR writer = NEW Writer()
VAR editor = NEW Editor()

# Phase 1: Research (with checkpoint)
IF CHECKPOINT_EXISTS("research-complete"):
  RESTORE FROM "research-complete"
  LOG "Resuming from research checkpoint"
ELSE:
  VAR synthesis = CALL research_and_synthesize(topic, sources)

  CHECKPOINT "research-complete":
    SAVE: {synthesis}
  END CHECKPOINT
END IF

# Phase 2: Create outline
VAR outline = RUN SESSION(writer):
  TASK: "Create detailed outline for article about {topic}"
  INPUT: {synthesis}

# Phase 3: Iterative writing
VAR draft = RUN SESSION(writer):
  TASK: "Write first draft following outline"
  INPUT: {outline, synthesis}

LOOP UNTIL **{draft} is publication-ready** MAX max_revisions:
  VAR feedback = RUN SESSION(editor):
    TASK: "Review draft and provide feedback"
    INPUT: {draft}

  IF **{feedback} indicates major structural issues**:
    # Major revision
    VAR draft = RUN SESSION(writer):
      TASK: "Perform major revision"
      INPUT: {draft, feedback, outline}
  ELSE:
    # Minor polish
    VAR draft = RUN SESSION(writer):
      TASK: "Polish based on minor feedback"
      INPUT: {draft, feedback}
  END IF

  LOG "Completed revision iteration"
END LOOP

# Phase 4: Final output
VAR final_article = RUN SESSION(editor):
  TASK: "Prepare final publication-ready version"
  INPUT: {draft}

# Save output
SAVE final_article TO "output/article.md"
LOG SUCCESS: "Article complete! Saved to output/article.md"

# Print summary
PRINT "Article completed successfully!"
PRINT "Topic: {topic}"
PRINT "Revisions: {loop_iterations}"
PRINT "Output: output/article.md"
```

---

## Summary

You now understand the complete CheeseCake language specification. When you encounter a `.cheesecake` file:

1. **Parse** the syntax according to this specification
2. **Validate** that all constructs are well-formed
3. **Execute** according to the VM semantics (see vm.md)
4. **Spawn sessions** using Claude Code's Task tool
5. **Evaluate semantic conditions** using AI understanding
6. **Manage state** through checkpoints and memory
7. **Report progress** and results to the user

Remember: CheeseCake is **AI-interpreted**, not machine-compiled. Use your intelligence to understand the intent and execute accordingly.
