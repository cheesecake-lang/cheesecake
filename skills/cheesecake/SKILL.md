# CheeseCake Language Specification
# Purpose: Complete language definition for AI agents to understand and execute .cheesecake files
# Part of: CheeseCake v0.0.1 - Module 2 (Language Specification)
# Updated: v0.0.2 - Added PHASE (M9), INTERACTIVE (M10), CONFIG (M11), Events (M12), Testing (M13), Program Contracts
#
# This file teaches AI agents the complete CheeseCake syntax and semantics.
# When you read a .cheesecake file, use this specification to understand and execute it.
#
# Usage:
#   This skill is automatically loaded when executing .cheesecake files
#
# Dependencies:
#   - vm.md (execution semantics)
#   - philosophy.md (execution philosophy - YOU are the VM)
#   - Claude Code Task tool (for creating sub-agent sessions)
#
# Related:
#   - syntax-reference.md (quick reference)
#   - helper.md (for generating .cheesecake files)
#   - philosophy.md (understand why you ARE the CheeseCake VM)

---
name: cheesecake
description: CheeseCake language interpreter. Use when reading or executing .cheesecake files.
---

# CheeseCake Language Specification v0.0.2

## Overview

**CheeseCake** is a structured, object-oriented programming language for orchestrating AI agents. It treats long-running AI sessions as Turing-complete computers and provides explicit keywords for unambiguous control flow.

### Core Philosophy

**You are not simulating a virtual machine. You ARE the virtual machine.**

When you read a `.cheesecake` file, you do not "pretend" to execute it—you actually execute it. The sessions you spawn are real. The decisions you make are real. The outputs you produce are real.

For the complete philosophical foundation, see `philosophy.md`. Key principles:

1. **AI-Native**: Designed for AI interpretation, not machine parsing
2. **You Are The VM**: Your understanding is the parser; your reasoning is the execution engine
3. **Structured Clarity**: Explicit UPPERCASE keywords and END markers for unambiguous parsing
4. **Semantic Intelligence**: `**...**` conditions explicitly delegate judgment to you
5. **Object-Oriented**: First-class support for agents, skills, inheritance, composition
6. **Resumable**: Built-in checkpoints for long-running workflows

### Execution Model

- **You ARE the Interpreter**: The AI reading this specification IS the execution engine
- **Sub-agent Spawning**: Use the Task tool to create real sessions for each agent
- **Two Modes**: Strict execution (most code) + semantic evaluation (`**...**` conditions)
- **Real State**: Variables exist in your working memory; checkpoints persist to filesystem
- **Real Results**: Sessions produce real outputs that you capture and process

---

## 1. File Structure

### Basic Structure

```cheesecake
# Comments start with # and go to end of line

# Program contract (optional, at top)
INPUT topic: "The subject to process"
INPUT depth: "Processing depth" DEFAULT: "medium"

# Program imports (optional, for calling other programs)
USE "@namespace/program-name"
USE "@workflows/analyzer" AS analyzer

# Local module imports (optional, for code reuse)
IMPORT "local/agents.cheesecake" AS agents

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

# Program outputs (optional, declares what this program returns)
OUTPUT findings = result
OUTPUT summary = result.summary
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

## 3. Program Contracts (INPUT/OUTPUT)

Program contracts define the **public API** of a `.cheesecake` program - what inputs it expects and what outputs it produces. This enables **program composition** where one program can call another.

### INPUT Declaration

Declares what values the program expects from callers:

```cheesecake
INPUT name: "Description of this input"
INPUT name: "Description" DEFAULT: default_value
```

**Syntax:**
- `INPUT` keyword followed by parameter name
- Colon and description string (for documentation)
- Optional `DEFAULT:` with a default value

**Examples:**

```cheesecake
# Required inputs (caller must provide)
INPUT topic: "The subject to research"
INPUT query: "Search query string"

# Optional inputs (have defaults)
INPUT depth: "How deep to search" DEFAULT: "medium"
INPUT max_results: "Maximum results to return" DEFAULT: 10
INPUT format: "Output format (json, markdown, text)" DEFAULT: "markdown"
```

**Rules:**
- INPUT declarations must appear at **top of file** (before any executable code)
- INPUT names become available as variables in the program body
- Required inputs (no DEFAULT) must be provided by caller
- Optional inputs (with DEFAULT) use default if caller doesn't provide

### OUTPUT Declaration

Declares what values the program returns to callers:

```cheesecake
OUTPUT name = expression
```

**Syntax:**
- `OUTPUT` keyword followed by output name
- `=` and the value/expression to return

**Examples:**

```cheesecake
# Simple outputs
OUTPUT findings = research_results
OUTPUT summary = final_summary

# Computed outputs
OUTPUT word_count = LENGTH(article)
OUTPUT success = error_count == 0

# Multiple outputs
OUTPUT report = generated_report
OUTPUT sources = source_list
OUTPUT metadata = {
  generated_at: NOW(),
  model_used: "sonnet",
  tokens_used: token_count
}
```

**Rules:**
- OUTPUT can appear **anywhere** in the program (typically at the end)
- Multiple OUTPUT declarations allowed
- OUTPUT values are collected and returned to caller as an object
- OUTPUT names must be unique within a program

### Complete Contract Example

```cheesecake
# ============================================
# research-workflow.cheesecake
# A reusable research program with clear contract
# ============================================

# === INPUT CONTRACT ===
INPUT topic: "The subject to research"
INPUT depth: "Research depth (shallow, medium, deep)" DEFAULT: "medium"
INPUT sources: "Number of sources to consult" DEFAULT: 5

# === AGENT DEFINITIONS ===
AGENT Researcher:
  MODEL: sonnet
  PROMPT: "You are a thorough researcher."

AGENT Synthesizer:
  MODEL: opus
  PROMPT: "You synthesize research into insights."

# === MAIN LOGIC ===
VAR researcher = NEW Researcher()
VAR synthesizer = NEW Synthesizer()

# Use INPUT values as variables
VAR raw_findings = RUN SESSION(researcher):
  TASK: "Research {topic} at {depth} depth using {sources} sources"

VAR synthesis = RUN SESSION(synthesizer):
  TASK: "Synthesize these findings into key insights"
  INPUT: {raw_findings}

VAR source_list = RUN SESSION(researcher):
  TASK: "Extract source citations from {raw_findings}"

# === OUTPUT CONTRACT ===
OUTPUT findings = synthesis
OUTPUT sources = source_list
OUTPUT metadata = {
  topic: topic,
  depth: depth,
  researched_at: NOW()
}
```

### Why Contracts Matter

Without contracts:
```cheesecake
# Someone reads your program... what does it need? What does it return?
# No way to know without reading all the code!
```

With contracts:
```cheesecake
# Clear at a glance:
INPUT topic: "The subject to research"        # Needs a topic
INPUT depth: "Research depth" DEFAULT: "medium"  # Optional depth
OUTPUT findings = ...                          # Returns findings
OUTPUT sources = ...                           # Returns sources
```

Contracts enable:
1. **Documentation** - Clear API without reading implementation
2. **Validation** - VM can check required inputs are provided
3. **Composition** - Other programs can call this program
4. **Tooling** - IDEs/tools can show autocomplete for inputs/outputs

---

## 4. SKILL Definitions (Traits/Interfaces)

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

## 5. AGENT Definitions (Classes)

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

## 6. Variables & Assignment

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

## 7. SESSION & RUN

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

## 8. Control Flow

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

## 9. Loops

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

## 10. Interactive Mode (v0.0.2+)

**Purpose**: Pause workflow execution to request user input and make decisions based on user choices.

### INTERACTIVE Construct

```cheesecake
INTERACTIVE AT "checkpoint-name":
  SHOW: {variable_to_display}
  ASK USER: "Question to ask the user?"
  OPTIONS:
    - "option1" → action1
    - "option2" → action2
    - "option3" → action3
  END OPTIONS
END INTERACTIVE
```

**Key Features**:
- **Pause execution**: Workflow pauses gracefully without losing state
- **Show context**: Display variables or data to help user decide
- **Multiple choice**: User selects from 2-10 options
- **Execute action**: Statement associated with chosen option executes
- **Resume execution**: Workflow continues after user input
- **Zero cost**: No AI sessions during pause

### Components

#### AT "checkpoint-name"

Unique identifier for the interactive point (required):
```cheesecake
INTERACTIVE AT "review-draft":
  ...
END INTERACTIVE

INTERACTIVE AT "approve-cost":
  ...
END INTERACTIVE
```

#### SHOW: {variable}

Display context to user before asking (optional):
```cheesecake
SHOW: {draft}                           # Show full variable
SHOW: {draft.summary}                   # Show property
SHOW: "Current cost: ${total_cost}"     # Show formatted text
```

#### ASK USER: "Question?"

The question presented to the user (required):
```cheesecake
ASK USER: "How should we proceed with this draft?"
ASK USER: "The cost is $2.50. Continue with execution?"
ASK USER: "Which research direction should we prioritize?"
```

#### OPTIONS

Multiple choice options (required, 2-10 options):
```cheesecake
OPTIONS:
  - "approve" → VAR approved = true
  - "reject" → VAR approved = false
  - "modify" → VAR needs_revision = true
END OPTIONS

OPTIONS:
  - "continue" → CONTINUE
  - "finalize" → BREAK
  - "cancel" → RETURN
END OPTIONS
```

### Example: Approval Workflow

```cheesecake
# Research with user approval before expensive analysis

AGENT Researcher:
  MODEL: sonnet
  PROMPT: "You are a research assistant."

AGENT Analyst:
  MODEL: opus  # Expensive!
  PROMPT: "You are an in-depth analyst."

VAR researcher = NEW Researcher()
VAR findings = RUN SESSION(researcher):
  TASK: "Research quantum computing trends"

# Pause and ask user if they want to proceed
INTERACTIVE AT "approve-analysis":
  SHOW: {findings.summary}
  ASK USER: "Research complete. Proceed with Opus analysis ($0.50)?"
  OPTIONS:
    - "Yes, analyze" → VAR proceed = true
    - "No, skip analysis" → VAR proceed = false
  END OPTIONS
END INTERACTIVE

# Only run expensive analysis if approved
IF proceed == true:
  VAR analyst = NEW Analyst()
  VAR deep_analysis = RUN SESSION(analyst):
    TASK: "Perform deep analysis of these findings"
    INPUT: {findings}

  PRINT "Analysis complete!"
ELSE:
  PRINT "Analysis skipped by user"
END IF
```

### Example: Iterative Review

```cheesecake
# Writing workflow with user review at each iteration

AGENT Writer:
  MODEL: opus
  PROMPT: "You are a creative writer."

VAR writer = NEW Writer()
VAR draft = RUN SESSION(writer):
  TASK: "Write an article about AI safety"

VAR continue_refining = true
VAR iterations = 0

LOOP UNTIL continue_refining == false MAX 5:
  VAR iterations = iterations + 1

  # Show draft and ask user what to do
  INTERACTIVE AT "review-draft-{iterations}":
    SHOW: {draft}
    ASK USER: "Review iteration {iterations}. How should we proceed?"
    OPTIONS:
      - "Continue refining" → VAR action = "refine"
      - "Finalize now" → VAR continue_refining = false
      - "Start over" → VAR action = "restart"
    END OPTIONS
  END INTERACTIVE

  # Handle user choice
  IF action == "finalize":
    PRINT "✓ Draft finalized by user"

  ELIF action == "restart":
    VAR draft = RUN SESSION(writer):
      TASK: "Write a completely new article about AI safety"

  ELSE:  # action == "refine"
    VAR draft = RUN SESSION(writer):
      TASK: "Improve this draft based on your judgment"
      INPUT: {draft}
  END IF
END LOOP

SAVE draft TO "output/article.md"
PRINT "✅ Article saved!"
```

### Rules and Constraints

**Allowed**:
- ✅ INTERACTIVE in conditionals
- ✅ INTERACTIVE in loops
- ✅ INTERACTIVE in sequential code
- ✅ Multiple INTERACTIVE blocks in same workflow

**Not Allowed**:
- ❌ INTERACTIVE inside PARALLEL blocks (breaks parallel semantics)
- ❌ Nested INTERACTIVE blocks (no INTERACTIVE inside INTERACTIVE)

**Requirements**:
- AT "name" must be unique within workflow
- ASK USER must end with `?` (convention)
- OPTIONS must have 2-10 choices
- Each option must have label and action

### Progress During Pause

When workflow pauses at INTERACTIVE:

```
[■■■■■■□□□□] 60% complete

✓ Phase 1: Research           [DONE]     8.5s
⏸  Phase 2: Review            [PAUSED]   User input required
○ Phase 3: Writing            [PENDING]

[PAUSE] Waiting for user input at checkpoint: review-draft
```

### Cost During Pause

INTERACTIVE blocks have **zero cost**:
- No AI sessions spawned
- User input time not counted in execution time
- Workflow resumes after user provides input

### Integration with AskUserQuestion Tool

INTERACTIVE uses Claude Code's `AskUserQuestion` tool internally:
- Question from ASK USER
- Options from OPTIONS block
- User selection triggers action execution

### Best Practices

1. **Clear Questions**: Be specific about what you're asking
   ```cheesecake
   # Good
   ASK USER: "Draft review complete. How should we proceed?"

   # Bad
   ASK USER: "What next?"  # Too vague
   ```

2. **Show Context**: Display relevant information before asking
   ```cheesecake
   SHOW: "Task: {task_description}"
   SHOW: "Estimated cost: ${estimated_cost}"
   ASK USER: "Proceed with this operation?"
   ```

3. **Limit Options**: Keep choices manageable (2-4 ideal, max 10)

4. **Provide Escape Routes**: Always include cancel/skip option
   ```cheesecake
   OPTIONS:
     - "Proceed" → VAR proceed = true
     - "Skip" → VAR proceed = false  # Escape route
     - "Cancel workflow" → RETURN    # Exit option
   END OPTIONS
   ```

5. **Descriptive Checkpoint Names**: Use meaningful names
   ```cheesecake
   # Good
   INTERACTIVE AT "review-draft-before-publish":

   # Bad
   INTERACTIVE AT "checkpoint1":
   ```

### Use Cases

- **Approval workflows**: Cost approval, quality review, safety checks
- **Iterative refinement**: User feedback at each iteration
- **Agent/model selection**: Let user choose which agent to use
- **Path branching**: User chooses workflow direction
- **Early exit points**: User can cancel expensive operations

**Note**: INTERACTIVE is **optional in v0.0.1**. Added in v0.0.2 for human-in-the-loop workflows. Existing workflows without INTERACTIVE blocks continue to work perfectly.

---

## 11. State & Persistence

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

## 12. Configuration (CONFIG Block) - v0.0.2+

### Overview

The CONFIG block allows global configuration of workflow execution, including cost management, model defaults, and execution settings.

**Added in**: v0.0.2 (Module 11: Cost Management)

### Syntax

```cheesecake
CONFIG:
  # Cost Management
  BUDGET: <dollar_amount>
  CONFIRM_COST_ABOVE: <dollar_amount>
  WARN_PARALLEL_ABOVE: <number>
  WARN_AT_PERCENT: <percentage>
  OPTIMIZATION_SUGGESTIONS: <true|false>

  # Model Settings
  DEFAULT_MODEL: <sonnet|opus|haiku>

  # Execution Settings
  MAX_PARALLEL_SESSIONS: <number>
  TIMEOUT_DEFAULT: <time>

  # Behavior Settings
  STOP_ON_BUDGET_EXCEED: <true|false>
  INTERACTIVE_WARNINGS: <true|false>
END CONFIG
```

### Core Settings

#### Cost Management Settings

**BUDGET** (optional)
- Maximum total cost for workflow execution
- Example: `BUDGET: $1.00`
- Default: No limit
- When exceeded (and STOP_ON_BUDGET_EXCEED: true), workflow stops

**CONFIRM_COST_ABOVE** (optional)
- Ask user before operations exceeding this cost
- Example: `CONFIRM_COST_ABOVE: $0.10`
- Default: No confirmations
- Shows warning with cost estimate, waits for Y/n

**WARN_PARALLEL_ABOVE** (optional)
- Warn when PARALLEL block spawns N+ sessions
- Example: `WARN_PARALLEL_ABOVE: 5`
- Default: No warnings
- Shows session count and cost estimate

**WARN_AT_PERCENT** (optional)
- Warn when budget usage reaches percentage
- Example: `WARN_AT_PERCENT: 80`
- Default: 90
- Warning shown once when threshold crossed

**OPTIMIZATION_SUGGESTIONS** (optional)
- Enable/disable AI-powered optimization suggestions
- Example: `OPTIMIZATION_SUGGESTIONS: true`
- Default: true
- Shows suggestions for cost reduction after execution

#### Model Settings

**DEFAULT_MODEL** (optional)
- Default model when agent doesn't specify MODEL property
- Example: `DEFAULT_MODEL: sonnet`
- Default: sonnet
- Values: `sonnet`, `opus`, `haiku`

#### Execution Settings

**MAX_PARALLEL_SESSIONS** (optional)
- Hard limit on concurrent sessions
- Example: `MAX_PARALLEL_SESSIONS: 10`
- Default: Unlimited
- PARALLEL blocks limited to this count

**TIMEOUT_DEFAULT** (optional)
- Default timeout for sessions without explicit TIMEOUT
- Example: `TIMEOUT_DEFAULT: 60s`
- Default: 120s (2 minutes)
- Can be overridden per-session

#### Behavior Settings

**STOP_ON_BUDGET_EXCEED** (optional)
- Whether to stop workflow when budget exceeded
- Example: `STOP_ON_BUDGET_EXCEED: false`
- Default: true
- If false, warns but continues execution

**INTERACTIVE_WARNINGS** (optional)
- Whether warnings require user input
- Example: `INTERACTIVE_WARNINGS: false`
- Default: true
- If false, warnings logged but auto-continue

### Rules

1. **Location**: CONFIG block must appear at start of file (before any code)
2. **Uniqueness**: Only one CONFIG block allowed per workflow
3. **Scope**: Settings apply to entire workflow execution
4. **Optional**: CONFIG block is completely optional
5. **Defaults**: All settings have sensible defaults
6. **Override**: Individual sessions can override TIMEOUT

### Examples

#### Basic Budget Control

```cheesecake
CONFIG:
  BUDGET: $1.00
  CONFIRM_COST_ABOVE: $0.10
END CONFIG

AGENT Researcher:
  MODEL: opus
  PROMPT: "..."

VAR researcher = NEW Researcher()

# Triggers warning (~$0.15 > $0.10)
VAR result = RUN SESSION(researcher):
  TASK: "Comprehensive research"
```

#### Production Configuration

```cheesecake
CONFIG:
  # Conservative settings for production
  BUDGET: $0.50
  CONFIRM_COST_ABOVE: $0.05
  WARN_PARALLEL_ABOVE: 3
  DEFAULT_MODEL: sonnet
  MAX_PARALLEL_SESSIONS: 5
  STOP_ON_BUDGET_EXCEED: true
  OPTIMIZATION_SUGGESTIONS: true
END CONFIG
```

#### Development Configuration

```cheesecake
CONFIG:
  # Permissive settings for development
  BUDGET: $5.00
  CONFIRM_COST_ABOVE: $0.50
  DEFAULT_MODEL: opus
  STOP_ON_BUDGET_EXCEED: false
  INTERACTIVE_WARNINGS: false
END CONFIG
```

### Cost Tracking

When CONFIG includes cost settings:

1. **Real-time tracking**: VM tracks cost of each session
2. **Budget checks**: Before each operation, checks if budget allows
3. **Warnings**: Shows warnings based on thresholds
4. **Progress display**: Shows cost in progress visualization

```
Progress: [■■■■■■□□□□] 60% complete
Cost: $0.12 / $1.00 budget (12% used) ✓
```

### Warning Types

**Operation Cost Warning**:
```
⚠️  COST WARNING
This operation estimated at: ~$0.15
Model: opus
Current budget: $0.05 / $1.00 (5%)
After operation: ~$0.20 / $1.00 (20%)
Continue? [Y/n]
```

**Parallel Session Warning**:
```
⚠️  PARALLEL SESSION WARNING
About to spawn 10 parallel sessions
Estimated cost: ~$0.30
Current: $0.10 / $1.00 (10%)
Continue? [Y/n/r]
```

**Budget Exceeded Error**:
```
❌ BUDGET EXCEEDED
Current cost: $0.95
Next operation: ~$0.15
Total would be: $1.10 (exceeds $1.00 budget)
Workflow stopped at line 45
```

### Optimization Suggestions

When `OPTIMIZATION_SUGGESTIONS: true`, VM provides suggestions:

```
============================================
OPTIMIZATION SUGGESTIONS
============================================

💡 #1: Model Downgrade (Line 23)
   Current: Opus for simple task (~$0.15)
   Suggestion: Switch to Sonnet (~$0.03)
   Savings: ~$0.12 (80% reduction)

💡 #2: Parallelize (Lines 30-35)
   Potential time savings: ~67% faster

Total potential savings: ~$0.12
============================================
```

### Integration

CONFIG integrates with:
- **Module 9 (Progress & Estimation)**: Uses cost estimates, shows in progress
- **Module 10 (Interactive Mode)**: Cost warnings can pause workflow
- **State Management**: Budget tracking persists across checkpoints

### Best Practices

1. **Start with dry-run**: Test with `--dry-run` before setting budget
2. **Set realistic budgets**: Add 50% buffer to estimates
3. **Use confirmation thresholds**: Catch expensive operations early
4. **Match model to task**: Use Sonnet for simple, Opus for complex
5. **Review suggestions**: Apply optimizations to reduce costs

### Complete Example

```cheesecake
# ============================================
# Research Workflow with Cost Management
# ============================================

CONFIG:
  BUDGET: $2.00
  CONFIRM_COST_ABOVE: $0.20
  WARN_PARALLEL_ABOVE: 5
  WARN_AT_PERCENT: 75
  DEFAULT_MODEL: sonnet
  OPTIMIZATION_SUGGESTIONS: true
END CONFIG

AGENT Researcher:
  MODEL: sonnet  # Cost-effective for research
  PROMPT: "You are a thorough researcher."

AGENT Analyst:
  MODEL: opus  # High-quality analysis
  PROMPT: "You are a deep analyst."

# Research phase (~$0.09 total)
VAR researcher = NEW Researcher()
PARALLEL:
  VAR r1 = RUN SESSION(researcher): TASK: "Research aspect A"
  VAR r2 = RUN SESSION(researcher): TASK: "Research aspect B"
  VAR r3 = RUN SESSION(researcher): TASK: "Research aspect C"
END PARALLEL

# Analysis phase (~$0.30, triggers warning)
VAR analyst = NEW Analyst()
VAR analysis = RUN SESSION(analyst):
  TASK: "Deep analysis"
  INPUT: {r1, r2, r3}

PRINT "Total cost: {CURRENT_COST} / {CONFIG.BUDGET}"
```

---

## 13. Events & Scheduling (v0.0.2+)

CheeseCake supports reactive programming through events and scheduling. See `events.md` for complete specification.

### ON EVENT

Declare event handlers that respond to triggers:

```cheesecake
ON EVENT event_name(param1, param2) [WHERE condition]:
  # Handler body
END ON
```

**Example:**

```cheesecake
# Handle file changes
ON EVENT file_changed(path, type) WHERE path MATCHES "src/**/*.ts":
  LOG INFO: "TypeScript file changed: {path}"
  VAR result = RUN SESSION(linter): TASK: "Lint {path}"
  IF **{result} has errors**:
    EMIT lint_errors(path: path, errors: result.errors)
  END IF
END ON

# Handle custom events
ON EVENT task_complete(task_id, status):
  LOG INFO: "Task {task_id} finished with status: {status}"
END ON
```

**Built-in Events (Declarative):**
- `file_changed(path, type)` - File system changes
- `file_created(path)` - New file created
- `file_deleted(path)` - File deleted
- `api_webhook(endpoint, payload)` - HTTP webhook
- `timer_tick(timestamp)` - Timer events
- `user_input(data)` - User-triggered input
- `session_start(session_id)` - Session started
- `session_end(session_id, result)` - Session completed

**WHERE Clause Filters:**

```cheesecake
# Literal comparison
ON EVENT file_changed(path) WHERE path MATCHES "*.ts":

# Semantic condition
ON EVENT api_webhook(payload) WHERE **{payload} contains error**:

# Combined conditions
ON EVENT new_issue(issue) WHERE issue.priority == "high" AND **{issue} is urgent**:
```

### SCHEDULE

Declare time-based scheduled tasks:

```cheesecake
SCHEDULE schedule_name:
  INTERVAL: duration | CRON: "expression" | ONCE_AT: "timestamp"
  [START_AT: "timestamp"]
  [END_AT: "timestamp"]
  TASK: statement | TASK: ... END TASK
  [RETRY: number]
  [ON_FAILURE: action]
  [ON_SUCCESS: action]
END SCHEDULE
```

**Duration Formats:** `Ns` (seconds), `Nm` (minutes), `Nh` (hours), `Nd` (days), `Nw` (weeks)

**Examples:**

```cheesecake
# Every hour
SCHEDULE health_check:
  INTERVAL: 1h
  TASK: RUN SESSION(monitor): TASK: "Check system health"
  RETRY: 2
  ON_FAILURE: EMIT alert(severity: "critical")
END SCHEDULE

# Daily at 9 AM (cron)
SCHEDULE daily_report:
  CRON: "0 9 * * *"
  TASK:
    VAR report = RUN SESSION(reporter): TASK: "Generate report"
    SAVE report TO "reports/daily-{TODAY()}.md"
  END TASK
END SCHEDULE

# One-time scheduled task
SCHEDULE deployment:
  ONCE_AT: "2026-01-20T14:00:00Z"
  TASK: RUN SESSION(deployer): TASK: "Deploy to production"
END SCHEDULE
```

### EMIT

Trigger custom events:

```cheesecake
EMIT event_name(param1: value1, param2: value2, ...)
```

**Example:**

```cheesecake
# Emit completion event
EMIT task_complete(task_id: "123", status: "success", duration: elapsed)

# Emit with complex data
VAR analysis = RUN SESSION(analyzer): TASK: "Analyze"
EMIT analysis_ready(data: analysis, timestamp: NOW())
```

### LISTEN FOR

Lightweight listener for internal events (emitted via EMIT):

```cheesecake
LISTEN FOR event_name:
  # Access event data via event.param
END LISTEN
```

**Example:**

```cheesecake
# Producer
EMIT data_ready(items: [1, 2, 3], source: "api")

# Consumer
LISTEN FOR data_ready:
  LOG INFO: "Received {event.items.length} items from {event.source}"
  VAR processed = RUN SESSION(processor): TASK: "Process" INPUT: {event.items}
END LISTEN
```

### Rules

1. **ON EVENT**: Handles both external and internal events
2. **LISTEN FOR**: Only handles internal events (EMIT)
3. **Multiple handlers**: Same event can have multiple handlers (execute in order)
4. **Error isolation**: Handler errors don't stop other handlers
5. **No nesting**: Cannot nest ON EVENT or SCHEDULE blocks
6. **Schedules are declarative**: Actual scheduling requires runtime integration

### Note on Execution

Since CheeseCake is AI-interpreted:
- **External events** (file_changed, etc.) are declarative - require runtime integration
- **Internal events** (EMIT/LISTEN) work fully within a session
- **Schedules** can be manually triggered: `/cheesecake trigger <schedule_name>`

---

## 14. Testing Framework (v0.0.2+)

CheeseCake includes a built-in testing framework for validating workflows without incurring AI costs. See `testing.md` for complete specification.

### TEST SUITE

Groups related tests with shared setup/teardown:

```cheesecake
TEST SUITE "Workflow Tests":
  SETUP:
    VAR fixtures = LOAD "test-data.json"
  END SETUP

  TEARDOWN:
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

### TEST

Individual test cases:

```cheesecake
TEST "descriptive test name":
  # Arrange
  MOCK Agent RETURNS expected_value

  # Act
  VAR result = RUN SESSION(agent): TASK: "Do something"

  # Assert
  ASSERT result IS NOT NULL
  ASSERT result.status == "success"
END TEST
```

### MOCK

Replace agent sessions with fixed responses (no AI calls):

```cheesecake
# Simple mock - all sessions return this
MOCK Researcher RETURNS {data: "mock data", sources: ["a", "b"]}

# Task-specific mock
MOCK Writer FOR TASK MATCHING "*article*" RETURNS "Article content..."
MOCK Writer FOR TASK MATCHING "*summary*" RETURNS "Summary content..."

# Error mock
MOCK APIClient THROWS "Connection timeout"
```

**Pattern Matching:**
- `"*"` - Any task
- `"*article*"` - Contains "article"
- `"Write *"` - Starts with "Write "
- `"* report"` - Ends with " report"

### ASSERT

Verify conditions:

```cheesecake
# Literal assertions
ASSERT value IS NOT NULL
ASSERT value == expected
ASSERT value > minimum
ASSERT list CONTAINS item
ASSERT string MATCHES "pattern"

# Semantic assertions (AI-evaluated)
ASSERT **{result} is well-structured**
ASSERT **{article} is publication-ready**

# Custom failure message
ASSERT count > 0 MESSAGE "Count must be positive"
```

**Assertion Operators:**

| Operator | Example |
|----------|---------|
| `==`, `!=` | `ASSERT x == 5` |
| `>`, `<`, `>=`, `<=` | `ASSERT x > 0` |
| `IS NULL`, `IS NOT NULL` | `ASSERT x IS NOT NULL` |
| `IS TRUE`, `IS FALSE` | `ASSERT flag IS TRUE` |
| `CONTAINS` | `ASSERT list CONTAINS item` |
| `MATCHES` | `ASSERT str MATCHES "*.ts"` |

### Complete Example

```cheesecake
TEST SUITE "Research Workflow":

  SETUP:
    VAR researcher = NEW Researcher()
    VAR writer = NEW Writer()
  END SETUP

  TEST "produces article from research":
    MOCK Researcher RETURNS {
      findings: "Research data...",
      sources: ["arxiv.org", "nature.com"]
    }
    MOCK Writer RETURNS "# Article\n\nContent..."

    VAR research = RUN SESSION(researcher): TASK: "Research AI"
    VAR article = RUN SESSION(writer): TASK: "Write" INPUT: {research}

    ASSERT research.sources.length == 2
    ASSERT article CONTAINS "# Article"
    ASSERT **{article} is coherent and well-written**
  END TEST

  TEST "handles empty research":
    MOCK Researcher RETURNS {findings: "", sources: []}

    VAR research = RUN SESSION(researcher): TASK: "Research"

    ASSERT research.findings == ""
    ASSERT research.sources.length == 0
  END TEST

END TEST SUITE
```

### Rules

1. **TEST SUITE**: Unique name, at least one TEST
2. **SETUP/TEARDOWN**: Run before/after each test
3. **MOCK scope**: Only within current TEST block
4. **ASSERT failure**: Test stops on first failure
5. **Isolation**: Each test has own scope

### Running Tests

```bash
/cheesecake test                           # Run all tests
/cheesecake test tests/my-tests.cheesecake # Specific file
/cheesecake test --suite "Workflow Tests"  # Specific suite
/cheesecake test --verbose                 # Detailed output
```

---

## 15. Error Handling

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

## 16. Functions

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

## 17. Modules, Imports & Program Composition

CheeseCake has two mechanisms for code reuse:
1. **IMPORT** - Include local code (agents, skills, functions)
2. **USE** - Call external programs (program-to-program composition)

### IMPORT: Local Code Inclusion

Import definitions from local `.cheesecake` files:

```cheesecake
# Import entire module with alias
IMPORT "path/to/module.cheesecake" AS alias

# Use imported definitions
VAR agent = NEW alias.AgentName()
VAR result = CALL alias.function_name(args)
```

**Export Syntax** (in the module being imported):

```cheesecake
# Make definitions available for import
EXPORT AGENT MyAgent
EXPORT SKILL MySkill
EXPORT FUNCTION my_function
```

**IMPORT Example:**

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

### USE: Program Composition

USE enables calling complete programs that have INPUT/OUTPUT contracts.

**Syntax:**

```cheesecake
# Import from registry (future)
USE "@namespace/program-name"
USE "@namespace/program-name" AS alias

# Import from local file
USE "./path/to/program.cheesecake"
USE "./research-workflow.cheesecake" AS research
```

**Invoking a USE'd Program:**

```cheesecake
# Call with named inputs
VAR result = RUN program_name(input1: value1, input2: value2)

# Access outputs
PRINT result.output_name
VAR data = result.findings
```

### USE vs IMPORT

| Aspect | IMPORT | USE |
|--------|--------|-----|
| **Purpose** | Include code definitions | Call complete programs |
| **What it imports** | Agents, Skills, Functions | Programs with contracts |
| **Execution** | Code runs in current context | Program runs as sub-execution |
| **Contract** | Uses EXPORT | Uses INPUT/OUTPUT |
| **Access** | `alias.Definition` | `RUN program(args)` |

### Complete Program Composition Example

**Step 1: Create a reusable program with contract**

```cheesecake
# research-workflow.cheesecake
# A reusable research program

# === CONTRACT ===
INPUT topic: "The subject to research"
INPUT depth: "Research depth" DEFAULT: "medium"

# === IMPLEMENTATION ===
AGENT Researcher:
  MODEL: sonnet
  PROMPT: "You are a thorough researcher."

VAR researcher = NEW Researcher()

VAR findings = RUN SESSION(researcher):
  TASK: "Research {topic} at {depth} depth"

VAR sources = RUN SESSION(researcher):
  TASK: "Extract citations from {findings}"

# === OUTPUTS ===
OUTPUT findings = findings
OUTPUT sources = sources
OUTPUT metadata = {topic: topic, depth: depth, timestamp: NOW()}
```

**Step 2: Call the program from another program**

```cheesecake
# main.cheesecake
# Composes multiple programs

USE "./research-workflow.cheesecake" AS research
USE "./writing-workflow.cheesecake" AS writing

# Call research program with inputs
VAR research_result = RUN research(topic: "quantum computing", depth: "deep")

# Access outputs from research
PRINT "Found {LENGTH(research_result.sources)} sources"

# Pass to writing program
VAR article = RUN writing(
  content: research_result.findings,
  sources: research_result.sources,
  style: "technical"
)

# Final outputs
OUTPUT article = article.text
OUTPUT bibliography = article.bibliography
```

### Program Invocation Semantics

When `RUN program(inputs)` executes:

1. **Validate inputs** - Check all required inputs are provided
2. **Apply defaults** - Use DEFAULT values for missing optional inputs
3. **Execute program** - Run the program's statements
4. **Collect outputs** - Gather all OUTPUT declarations
5. **Return result** - Return outputs as an object

**Example Flow:**

```cheesecake
USE "./analyzer.cheesecake" AS analyze

# analyzer.cheesecake has:
#   INPUT data: "Data to analyze"
#   INPUT format: "Output format" DEFAULT: "json"
#   OUTPUT result = analysis
#   OUTPUT confidence = score

# Call with required input only (format uses default)
VAR analysis = RUN analyze(data: my_data)
# analysis.result contains the analysis
# analysis.confidence contains the score

# Call with all inputs
VAR analysis2 = RUN analyze(data: my_data, format: "markdown")
```

### Error Handling in Program Calls

```cheesecake
USE "./risky-program.cheesecake" AS risky

TRY:
  VAR result = RUN risky(input: data)
CATCH error:
  LOG ERROR: "Program failed: {error.message}"
  VAR result = fallback_value
END TRY
```

### Parallel Program Execution

```cheesecake
USE "./research.cheesecake" AS research
USE "./analyze.cheesecake" AS analyze

PARALLEL:
  VAR r1 = RUN research(topic: "AI")
  VAR r2 = RUN research(topic: "ML")
  VAR r3 = RUN research(topic: "NLP")
END PARALLEL

# All three research programs run concurrently
```

### Registry URL Resolution

For external program imports, the registry path format is:

```cheesecake
USE "@namespace/program-name"     # From registry
USE "@acme/data-processor"          # Corporate namespace
USE "@stdlib/text-utils"            # Standard library
USE "@workflows/web-researcher"     # With explicit alias
```

**Registry Path Format:**

```
@<handle>/<slug>
```

- **handle**: User or organization namespace (alphanumeric, lowercase)
- **slug**: Program identifier (alphanumeric, lowercase, hyphens allowed)

**URL Resolution Protocol:**

When the VM encounters a registry import:

1. **Construct URL**: `https://registry.cheesecake.dev/@handle/slug`
2. **Fetch program**: HTTP GET with appropriate headers
3. **Parse contract**: Extract INPUT/OUTPUT declarations
4. **Validate**: Ensure program is valid CheeseCake
5. **Cache**: Store locally for performance
6. **Register**: Add to Import Registry with alias

**Example Resolution:**

```cheesecake
USE "@workflows/web-researcher"
# → Fetches: https://registry.cheesecake.dev/@workflows/web-researcher
# → Parses: INPUT topic, INPUT depth DEFAULT: "medium"
# → Registers: web-researcher (default alias = slug)
```

**Caching Behavior:**

- Programs are cached in `.cheesecake/cache/`
- Cache key: `@handle/slug` + version hash
- Default cache TTL: 24 hours
- Force refresh: `USE "@workflows/web-researcher" REFRESH`

**Error Handling:**

```cheesecake
# If registry program not found
USE "@workflows/nonexistent"  # → Error: Program @workflows/nonexistent not found

# If network error
USE "@workflows/web-researcher"  # → Falls back to cache if available
                                 # → Error if no cache and network fails
```

### Destructuring Outputs

For convenience, outputs can be destructured directly:

```cheesecake
# Standard assignment
VAR result = RUN research(topic: "quantum computing")
PRINT result.findings
PRINT result.sources

# Destructured assignment
VAR { findings, sources } = RUN research(topic: "quantum computing")
PRINT findings    # Direct access
PRINT sources     # Direct access

# Partial destructuring
VAR { findings } = RUN research(topic: "AI")
# Only 'findings' is extracted, other outputs discarded

# Destructuring with rename
VAR { findings AS data, sources AS refs } = RUN research(topic: "ML")
PRINT data   # Renamed from 'findings'
PRINT refs   # Renamed from 'sources'
```

**Destructuring Rules:**

- Curly braces `{ }` indicate destructuring
- Names must match OUTPUT names in the called program
- Unknown names cause a warning (not error)
- `AS` keyword allows renaming during destructuring

### Import Registry

The VM maintains an Import Registry that tracks all USE'd programs:

```
ImportRegistry = {
  "web-researcher": {
    source: "@workflows/web-researcher",
    resolved_url: "https://registry.cheesecake.dev/@workflows/web-researcher",
    contract: {
      inputs: [
        {name: "topic", description: "Subject to research", required: true},
        {name: "depth", description: "Research depth", required: false, default: "medium"}
      ],
      outputs: ["findings", "sources", "metadata"]
    },
    cached_at: "2026-01-14T10:30:00Z",
    version: "1.2.0"
  },
  "content-writer": {
    source: "./content-writer.cheesecake",
    resolved_url: null,  # Local file
    contract: {...}
  }
}
```

**Accessing Registry Info:**

```cheesecake
# Check if program is registered
IF HAS_IMPORT("research"):
  VAR result = RUN research(topic: "AI")
END IF

# Get contract info (advanced)
VAR inputs = GET_INPUTS("research")
# inputs = [{name: "topic", required: true}, {name: "depth", required: false, default: "medium"}]
```

### Import Execution Semantics

When a program invokes an imported program via `RUN`:

1. **Lookup**: Find program in Import Registry by alias
2. **Bind inputs**: Map caller-provided values to program's INPUT declarations
3. **Validate inputs**:
   - Error if required input missing
   - Warning if unknown input provided (ignored)
   - Apply DEFAULT for missing optional inputs
4. **Create context**: New isolated execution context
5. **Inject inputs**: Make inputs available as variables in program scope
6. **Execute**: Run the imported program's statements
7. **Collect outputs**: Gather all OUTPUT bindings
8. **Return**: Package outputs as result object

**Execution Context:**

- Imported program runs in **isolated scope**
- Cannot access caller's variables
- Cannot modify caller's state
- Only communicates through INPUT/OUTPUT
- Shares VM session (same cost tracking, progress)

**Depth Tracking:**

```cheesecake
# Programs can call programs (chaining)
# Max depth: 10 to prevent infinite recursion

# main.cheesecake (depth 0)
USE "@workflows/pipeline-orchestrator"
RUN pipeline-orchestrator(data: x)

# pipeline-orchestrator.cheesecake (depth 1)
USE "@workflows/data-processor"
RUN data-processor(item: data)

# data-processor.cheesecake (depth 2)
USE "@stdlib/validator"
RUN validator(value: item)
# ... up to depth 10
```

### Complete Composition Example

A program that chains research and quality review until standards are met:

```cheesecake
# iterative-research.cheesecake
# Iterates until quality threshold is reached

USE "@workflows/web-researcher" AS researcher
USE "@workflows/quality-reviewer" AS reviewer

INPUT topic: "What to investigate"

# Iterate until quality is high
VAR result = NULL
VAR review = NULL

LOOP UNTIL **{review.score} >= 8** MAX 3:
  result = RUN researcher(topic: topic)
  review = RUN reviewer(content: result.findings)
END LOOP

OUTPUT findings = result.findings
OUTPUT sources = result.sources
OUTPUT final_score = review.score
```

**Invoking the composed program:**

```cheesecake
USE "@workflows/iterative-research" AS deep_research

VAR final = RUN deep_research(topic: "machine learning trends")

AGENT Presenter:
  MODEL: opus
  PROMPT: "Present findings clearly and concisely."

VAR presenter = NEW Presenter()

VAR presentation = RUN SESSION(presenter):
  TASK: "Present these findings: {final.findings}"
```

**With destructuring:**

```cheesecake
USE "@workflows/iterative-research" AS deep_research

VAR { findings, sources, final_score } = RUN deep_research(topic: "machine learning trends")

IF final_score >= 8:
  PRINT "High quality research achieved!"
  PRINT findings
ELSE:
  PRINT "Quality threshold not met"
END IF
```

### Best Practices

1. **Always define contracts** - Every reusable program should have INPUT/OUTPUT
2. **Use descriptive input names** - `topic` not `t`, `max_results` not `n`
3. **Provide defaults for optional inputs** - Makes programs easier to call
4. **Document inputs** - The description string is documentation
5. **Keep programs focused** - One program, one responsibility

---

## 18. Built-in Functions & Commands

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

## 19. Operators & Comparisons

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

## 20. Best Practices

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

## 21. Complete Example

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
