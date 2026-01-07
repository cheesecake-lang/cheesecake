# CheeseCake Interactive Mode Specification
# Purpose: Define how workflows can pause for user input and make decisions
# Part of: CheeseCake v0.0.2 - Module 10 (Interactive Mode)
#
# This file specifies how CheeseCake workflows can pause execution to:
# - Show current state to the user
# - Ask questions and get input
# - Offer multiple choice options
# - Branch workflow based on user decisions
#
# Usage:
#   Referenced by vm.md for interactive execution behavior
#   Used when INTERACTIVE blocks are encountered
#
# Dependencies:
#   - vm.md (execution semantics)
#   - AskUserQuestion tool (Claude Code built-in)
#   - SKILL.md (language specification)
#
# Related:
#   - progress.md (progress tracking continues during pauses)
#   - cost-estimation.md (pauses don't incur cost)

---

## Overview

Interactive Mode allows workflows to **pause execution** and request user input, enabling:
- **Human-in-the-loop workflows** where user judgment guides AI actions
- **Decision points** where users choose the next path
- **Review checkpoints** where users approve before expensive operations
- **Iterative refinement** where users give feedback mid-workflow

### Core Principles

1. **Non-blocking**: Workflows pause gracefully without losing state
2. **Clear presentation**: Users see exactly what they need to decide
3. **Type-safe**: Options and inputs are validated
4. **Resumable**: Execution continues seamlessly after input
5. **Cost-aware**: Pauses don't incur AI costs

---

## INTERACTIVE Construct

### Basic Syntax

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

### Components

#### 1. AT "checkpoint-name"

**Purpose**: Unique identifier for this interactive point

**Rules**:
- Must be a **unique string** within the workflow
- Used for logging, debugging, and resume
- Descriptive names recommended (e.g., "review-draft", "approve-cost")

**Example**:
```cheesecake
INTERACTIVE AT "review-checkpoint":
  ...
END INTERACTIVE
```

#### 2. SHOW: {variable}

**Purpose**: Display current state/data to the user before asking for input

**Rules**:
- Optional (can omit if no context needed)
- Can show any variable or expression
- Supports objects, strings, arrays
- VM formats the display appropriately

**Examples**:
```cheesecake
SHOW: {draft}                          # Show full draft text
SHOW: {draft.summary}                  # Show summary property
SHOW: {analysis}                       # Show analysis results
SHOW: "Current cost: ${total_cost}"    # Show formatted text
```

#### 3. ASK USER: "Question?"

**Purpose**: The question or prompt presented to the user

**Rules**:
- Required (must ask a clear question)
- Should end with `?` for clarity
- Keep concise but descriptive
- Use context from SHOW if needed

**Examples**:
```cheesecake
ASK USER: "How should we proceed with this draft?"
ASK USER: "The cost is $2.50. Continue with execution?"
ASK USER: "Which research direction should we prioritize?"
```

#### 4. OPTIONS

**Purpose**: Multiple choice options the user can select from

**Rules**:
- Minimum **2 options**, maximum **10 options**
- Each option has a **label** and an **action**
- Actions can be:
  - Variable assignments
  - Session executions
  - Control flow (BREAK, CONTINUE, RETURN)
  - Any valid CheeseCake statements

**Syntax**:
```cheesecake
OPTIONS:
  - "label1" → statement1
  - "label2" → statement2
  - "label3" → statement3
END OPTIONS
```

**Examples**:
```cheesecake
OPTIONS:
  - "approve" → VAR approved = true
  - "reject" → VAR approved = false
  - "modify" → VAR needs_revision = true
END OPTIONS

OPTIONS:
  - "continue" → CONTINUE
  - "finalize" → BREAK
  - "edit manually" → VAR manual_edit = true
END OPTIONS

OPTIONS:
  - "use opus" → VAR model = "opus"
  - "use sonnet" → VAR model = "sonnet"
  - "skip this task" → CONTINUE
END OPTIONS
```

---

## Execution Semantics

### How INTERACTIVE Works

When the VM encounters an `INTERACTIVE` block:

#### Step 1: Pause Execution

- **Save state**: Current variables, progress, context
- **Mark pause point**: Record checkpoint name and line number
- **Preserve context**: All session data remains available

#### Step 2: Display Context

If `SHOW` is present:
- Format the variable/expression for display
- Present to user in readable format
- Truncate if very long (show summary + "see full output")

#### Step 3: Present Question and Options

Using `AskUserQuestion` tool:
- Display the question from `ASK USER`
- Show all options as selectable choices
- Allow user to select one option
- Optionally allow "Other" for custom text input

#### Step 4: Execute Selected Action

- Execute the statement(s) associated with chosen option
- Update variables as needed
- Apply control flow (BREAK, CONTINUE, etc.)

#### Step 5: Resume Execution

- Continue with next statement after `END INTERACTIVE`
- Progress tracking resumes
- Workflow proceeds normally

### State During Pause

**Preserved**:
- All variables and their values
- Session contexts
- Checkpoint data
- Progress state

**Not Preserved**:
- Active AI sessions (they complete before pause)
- Timers (execution time pauses)

---

## Examples

### Example 1: Simple Approval

```cheesecake
# Research workflow with user approval before expensive analysis

AGENT Researcher:
  MODEL: sonnet
  PROMPT: "You are a research assistant."

AGENT Analyst:
  MODEL: opus  # Expensive!
  PROMPT: "You are an in-depth analyst."

VAR researcher = NEW Researcher()
VAR findings = RUN SESSION(researcher):
  TASK: "Research quantum computing trends"

# Pause and ask user if they want to proceed with expensive analysis
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

**User Experience**:
```
Running: research-workflow.cheesecake

✓ Research complete (8.5s)

[PAUSE] User input required

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Checkpoint: approve-analysis

Current findings summary:
"Quantum computing shows rapid advances in error correction
and qubit stability. Major investments from tech giants..."

Question: Research complete. Proceed with Opus analysis ($0.50)?

  [1] Yes, analyze
  [2] No, skip analysis

Your choice: _
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### Example 2: Iterative Draft Review

```cheesecake
# Writing workflow with user review at each iteration

AGENT Writer:
  MODEL: opus
  PROMPT: "You are a creative writer."

VAR writer = NEW Writer()
VAR draft = RUN SESSION(writer):
  TASK: "Write an article about AI safety"

VAR iterations = 0
VAR continue_refining = true

LOOP UNTIL continue_refining == false MAX 5:
  VAR iterations = iterations + 1

  # Show draft and ask user what to do
  INTERACTIVE AT "review-draft-iteration-{iterations}":
    SHOW: {draft}
    ASK USER: "Review iteration {iterations}. How should we proceed?"
    OPTIONS:
      - "Continue refining" → VAR action = "refine"
      - "Finalize now" → VAR action = "finalize"
      - "Start over" → VAR action = "restart"
      - "Abandon" → VAR action = "abandon"
    END OPTIONS
  END INTERACTIVE

  # Handle user choice
  IF action == "finalize":
    VAR continue_refining = false
    PRINT "✓ Draft finalized by user"

  ELIF action == "restart":
    VAR draft = RUN SESSION(writer):
      TASK: "Write a completely new article about AI safety"
    PRINT "→ Starting fresh draft"

  ELIF action == "abandon":
    VAR continue_refining = false
    VAR draft = NULL
    PRINT "✗ Draft abandoned by user"

  ELSE:  # action == "refine"
    # Get feedback and improve
    VAR draft = RUN SESSION(writer):
      TASK: "Improve this draft based on your judgment"
      INPUT: {draft}
    PRINT "→ Draft refined (iteration {iterations})"
  END IF
END LOOP

IF draft != NULL:
  SAVE draft TO "output/article.md"
  PRINT "✅ Article saved!"
ELSE:
  PRINT "No article produced."
END IF
```

**User Experience**:
```
Running: iterative-writing.cheesecake

✓ Initial draft complete (12.3s)

[PAUSE] User input required

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Checkpoint: review-draft-iteration-1

Current draft:
# Navigating the Future of AI Safety

Artificial intelligence systems are becoming...
[2,340 words]

Question: Review iteration 1. How should we proceed?

  [1] Continue refining
  [2] Finalize now
  [3] Start over
  [4] Abandon

Your choice: 1
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

→ Draft refined (iteration 1)

✓ Refinement complete (10.2s)

[PAUSE] User input required
...
```

---

### Example 3: Multi-Agent Selection

```cheesecake
# User chooses which agent to use based on task complexity

AGENT JuniorDev:
  MODEL: sonnet
  PROMPT: "You are a junior developer."

AGENT SeniorDev:
  MODEL: sonnet
  PROMPT: "You are a senior developer."

AGENT Architect:
  MODEL: opus
  PROMPT: "You are a software architect."

VAR task_description = "Build a user authentication system with OAuth2"

# Let user decide complexity level
INTERACTIVE AT "choose-developer":
  SHOW: "Task: {task_description}"
  ASK USER: "Which level of developer should handle this task?"
  OPTIONS:
    - "Junior (fast, simple)" → VAR choice = "junior"
    - "Senior (balanced)" → VAR choice = "senior"
    - "Architect (thorough)" → VAR choice = "architect"
  END OPTIONS
END INTERACTIVE

# Execute with chosen agent
IF choice == "junior":
  VAR dev = NEW JuniorDev()
  VAR implementation = RUN SESSION(dev): TASK: "{task_description}"

ELIF choice == "senior":
  VAR dev = NEW SeniorDev()
  VAR implementation = RUN SESSION(dev): TASK: "{task_description}"

ELSE:  # architect
  VAR arch = NEW Architect()

  # Architect does design first, then implementation
  VAR design = RUN SESSION(arch):
    TASK: "Design architecture for: {task_description}"

  VAR implementation = RUN SESSION(arch):
    TASK: "Implement based on this design"
    INPUT: {design}
END IF

PRINT "✓ Implementation complete!"
```

---

### Example 4: Cost-Aware Branching

```cheesecake
# Ask user to approve expensive operation with cost preview

IMPORT "cost-estimation.md" AS cost

VAR expensive_task = "Perform comprehensive market analysis across 50 companies"

# Estimate cost before execution
VAR estimated_cost = ESTIMATE TASK(expensive_task)

# Show cost and ask for approval
INTERACTIVE AT "cost-approval":
  SHOW: "Task: {expensive_task}"
  SHOW: "Estimated cost: ${estimated_cost}"
  ASK USER: "This operation will cost ~${estimated_cost}. Proceed?"
  OPTIONS:
    - "Yes, proceed" → VAR approved = true
    - "No, use cheaper alternative" → VAR approved = false
    - "Modify task scope" → VAR modify = true
  END OPTIONS
END INTERACTIVE

IF modify == true:
  # Let user modify the task
  INTERACTIVE AT "modify-task":
    ASK USER: "Enter modified task description:"
    OPTIONS:
      - "Analyze top 10 companies only" → VAR expensive_task = "Analyze top 10 companies"
      - "Focus on one industry" → VAR expensive_task = "Analyze fintech sector only"
      - "Cancel entirely" → VAR approved = false
    END OPTIONS
  END INTERACTIVE
END IF

IF approved == true:
  VAR analyst = NEW MarketAnalyst()
  VAR analysis = RUN SESSION(analyst):
    TASK: "{expensive_task}"

  PRINT "✓ Analysis complete!"
ELSE:
  PRINT "Operation cancelled by user"
END IF
```

---

### Example 5: Parallel Task Approval

```cheesecake
# Let user select which parallel tasks to run

VAR available_tasks = [
  {name: "Market research", cost: 0.05, time: "~10s"},
  {name: "Competitor analysis", cost: 0.08, time: "~15s"},
  {name: "Trend forecasting", cost: 0.12, time: "~20s"},
  {name: "Customer sentiment", cost: 0.06, time: "~12s"}
]

VAR selected_tasks = []

# Show all options and let user select multiple
INTERACTIVE AT "select-tasks":
  SHOW: {available_tasks}
  ASK USER: "Select which research tasks to run (can select multiple):"
  OPTIONS:
    - "All tasks (total: $0.31)" → VAR selected_tasks = available_tasks
    - "Market + Competitor ($0.13)" → VAR selected_tasks = [available_tasks[0], available_tasks[1]]
    - "Only Market research ($0.05)" → VAR selected_tasks = [available_tasks[0]]
    - "None (skip research)" → VAR selected_tasks = []
  END OPTIONS
END INTERACTIVE

IF selected_tasks.length > 0:
  VAR researcher = NEW Researcher()

  # Run selected tasks in parallel
  VAR results = []
  PARALLEL FOR task IN selected_tasks:
    VAR result = RUN SESSION(researcher): TASK: "{task.name}"
    results.APPEND(result)
  END PARALLEL FOR

  PRINT "✓ Completed {selected_tasks.length} tasks!"
ELSE:
  PRINT "No tasks selected"
END IF
```

---

## Integration with AskUserQuestion Tool

The INTERACTIVE construct uses Claude Code's `AskUserQuestion` tool internally.

### Mapping

```
INTERACTIVE construct         →  AskUserQuestion tool
─────────────────────────────────────────────────────
AT "checkpoint-name"          →  (Used for logging/context)
SHOW: {variable}              →  Displayed before question
ASK USER: "Question?"         →  question parameter
OPTIONS:                      →  options array
  - "label" → action          →  {label: "label", description: "..."}
```

### Implementation

When VM encounters `INTERACTIVE`:

```javascript
// Pseudo-code for VM implementation

function execute_interactive(block):
  // Step 1: Display context (if SHOW present)
  if block.has_show:
    context_text = evaluate_and_format(block.show_expression)
    display_to_user(context_text)

  // Step 2: Build options for AskUserQuestion
  options = []
  for option in block.options:
    options.append({
      label: option.label,
      description: explain_action(option.action)
    })

  // Step 3: Ask user using AskUserQuestion tool
  user_choice = AskUserQuestion(
    question: block.ask_user_text,
    header: block.checkpoint_name,
    options: options,
    multiSelect: false  // Single choice only
  )

  // Step 4: Find matching option and execute action
  selected_option = find_option_by_label(user_choice, block.options)
  execute_statement(selected_option.action)

  // Step 5: Continue execution
  return CONTINUE
```

---

## Advanced Features

### Nested INTERACTIVE Blocks

**NOT ALLOWED**: Cannot nest INTERACTIVE blocks

```cheesecake
# ❌ INVALID - nested INTERACTIVE
INTERACTIVE AT "outer":
  INTERACTIVE AT "inner":  # NOT ALLOWED
    ...
  END INTERACTIVE
END INTERACTIVE
```

**Workaround**: Use sequential INTERACTIVE blocks

```cheesecake
# ✅ VALID - sequential INTERACTIVE blocks
INTERACTIVE AT "first-choice":
  ASK USER: "Choose path?"
  OPTIONS:
    - "Path A" → VAR path = "A"
    - "Path B" → VAR path = "B"
  END OPTIONS
END INTERACTIVE

IF path == "A":
  INTERACTIVE AT "path-a-options":
    ASK USER: "Path A selected. Next step?"
    OPTIONS:
      - "Continue" → VAR proceed = true
      - "Go back" → VAR proceed = false
    END OPTIONS
  END INTERACTIVE
END IF
```

### Conditional INTERACTIVE

INTERACTIVE blocks can be inside conditionals:

```cheesecake
IF **{draft} needs significant improvement**:
  # Only ask user if draft quality is low
  INTERACTIVE AT "low-quality-draft":
    SHOW: {draft}
    ASK USER: "Draft quality is low. What should we do?"
    OPTIONS:
      - "Revise extensively" → VAR action = "revise"
      - "Start over" → VAR action = "restart"
      - "Accept as-is" → VAR action = "accept"
    END OPTIONS
  END INTERACTIVE
ELSE:
  # Skip user interaction if draft is good
  PRINT "Draft quality is acceptable, continuing..."
END IF
```

### INTERACTIVE in Loops

Useful for iterative workflows:

```cheesecake
VAR continue = true
VAR count = 0

LOOP UNTIL continue == false MAX 10:
  VAR count = count + 1

  # Do some work
  VAR result = RUN SESSION(agent): TASK: "Process item {count}"

  # Ask user after each iteration
  INTERACTIVE AT "loop-checkpoint-{count}":
    SHOW: {result}
    ASK USER: "Item {count} processed. Continue?"
    OPTIONS:
      - "Yes, continue" → CONTINUE
      - "No, stop here" → VAR continue = false
    END OPTIONS
  END INTERACTIVE
END LOOP
```

### INTERACTIVE with PARALLEL

**Rule**: INTERACTIVE cannot be inside PARALLEL block

```cheesecake
# ❌ INVALID - INTERACTIVE inside PARALLEL
PARALLEL:
  VAR result1 = RUN SESSION(agent1): TASK: "..."

  INTERACTIVE AT "choice":  # NOT ALLOWED in PARALLEL
    ASK USER: "Choose option?"
    OPTIONS: ...
  END INTERACTIVE
END PARALLEL
```

**Reason**: PARALLEL blocks execute concurrently; pausing one branch would break parallel semantics.

**Workaround**: Use INTERACTIVE before or after PARALLEL

```cheesecake
# ✅ VALID - INTERACTIVE before PARALLEL
INTERACTIVE AT "select-tasks":
  ASK USER: "Which tasks to run in parallel?"
  OPTIONS:
    - "Task A + B" → VAR tasks = ["A", "B"]
    - "Task A + C" → VAR tasks = ["A", "C"]
  END OPTIONS
END INTERACTIVE

PARALLEL:
  FOR task IN tasks:
    VAR result = RUN SESSION(agent): TASK: "{task}"
  END FOR
END PARALLEL
```

---

## Progress Tracking with INTERACTIVE

### Progress Display During Pause

When workflow pauses at INTERACTIVE:

```
[■■■■■■□□□□] 60% complete

✓ Phase 1: Research           [DONE]     8.5s
✓ Phase 2: Analysis           [DONE]     4.2s
⏸  Phase 3: Writing           [PAUSED]   User input required
○ Phase 4: Output             [PENDING]

[PAUSE] Waiting for user input at checkpoint: review-draft
```

### Time Tracking

- Execution time **pauses** during INTERACTIVE
- Time spent waiting for user input is **not counted**
- Time tracking resumes after user provides input

---

## Cost Tracking with INTERACTIVE

### Zero Cost During Pause

- INTERACTIVE blocks **do not spawn AI sessions**
- User input pauses **do not incur any cost**
- Cost estimation counts INTERACTIVE as zero cost

### Example Cost Estimate

```cheesecake
PHASE "Research":
  VAR findings = RUN SESSION(researcher): TASK: "..."  # $0.02
END PHASE

INTERACTIVE AT "review":
  SHOW: {findings}
  ASK USER: "Proceed with analysis?"
  OPTIONS:
    - "Yes" → VAR proceed = true
    - "No" → VAR proceed = false
  END OPTIONS
END INTERACTIVE  # $0.00 - no cost

PHASE "Analysis":
  IF proceed:
    VAR analysis = RUN SESSION(analyst): TASK: "..."  # $0.05
  END IF
END PHASE
```

**Cost Estimate**:
```
Phase 1: Research       $0.02
Interactive pause       $0.00  (no cost)
Phase 2: Analysis       $0.05 (if user selects "Yes")

Total: $0.02 - $0.07 (depends on user choice)
```

---

## Error Handling

### Missing User Input

If user doesn't provide input (session ends, timeout, etc.):

**Behavior**: Workflow **suspends** and saves checkpoint

**Resume**: User can resume workflow later with `/cheesecake resume`

### Invalid Option Selected

**Shouldn't happen**: `AskUserQuestion` tool ensures valid selection

**Fallback**: If somehow invalid, VM treats as first option

### Action Execution Fails

If the statement associated with an option fails:

```cheesecake
INTERACTIVE AT "choice":
  ASK USER: "Select option?"
  OPTIONS:
    - "Option A" → VAR x = RUN SESSION(agent): TASK: "..."  # Could fail
    - "Option B" → VAR x = NULL
  END OPTIONS
END INTERACTIVE

# If Option A selected but session fails:
TRY:
  # Action executes here
CATCH error:
  LOG ERROR: "Interactive option action failed: {error}"
  # Workflow continues or aborts based on error handling
END TRY
```

---

## Best Practices

### 1. Clear Questions

**Good**:
```cheesecake
ASK USER: "Draft review complete. How should we proceed?"
OPTIONS:
  - "Continue refinement" → VAR action = "refine"
  - "Finalize and save" → VAR action = "finalize"
END OPTIONS
```

**Bad**:
```cheesecake
ASK USER: "What next?"  # Too vague
OPTIONS:
  - "Do it" → VAR x = true  # Unclear what "it" is
  - "Don't" → VAR x = false
END OPTIONS
```

### 2. Show Relevant Context

**Good**:
```cheesecake
INTERACTIVE AT "approve-cost":
  SHOW: "Task: {task_description}"
  SHOW: "Estimated cost: ${estimated_cost}"
  SHOW: "Estimated time: {estimated_time}"
  ASK USER: "Proceed with this operation?"
  OPTIONS: ...
END INTERACTIVE
```

**Bad**:
```cheesecake
INTERACTIVE AT "approve":
  # No context shown - user doesn't know what they're approving
  ASK USER: "Approve?"
  OPTIONS: ...
END INTERACTIVE
```

### 3. Limit Options

- **2-4 options**: Ideal (easy to choose)
- **5-7 options**: Acceptable (more complex choice)
- **8-10 options**: Maximum (avoid if possible)

**Good**:
```cheesecake
OPTIONS:
  - "Approve" → VAR approved = true
  - "Reject" → VAR approved = false
  - "Modify first" → VAR needs_modification = true
END OPTIONS  # 3 options - clear choice
```

**Bad**:
```cheesecake
OPTIONS:
  - "Option 1" → ...
  - "Option 2" → ...
  - "Option 3" → ...
  - "Option 4" → ...
  - "Option 5" → ...
  - "Option 6" → ...
  - "Option 7" → ...
  - "Option 8" → ...
END OPTIONS  # 8 options - overwhelming
```

### 4. Use Descriptive Checkpoint Names

**Good**:
```cheesecake
INTERACTIVE AT "review-draft-before-publish":
  ...
END INTERACTIVE

INTERACTIVE AT "select-model-for-analysis":
  ...
END INTERACTIVE
```

**Bad**:
```cheesecake
INTERACTIVE AT "checkpoint1":  # Non-descriptive
  ...
END INTERACTIVE

INTERACTIVE AT "pause":  # Too generic
  ...
END INTERACTIVE
```

### 5. Provide Escape Routes

Always include an option to cancel, skip, or go back:

**Good**:
```cheesecake
OPTIONS:
  - "Proceed with analysis" → VAR proceed = true
  - "Skip analysis" → VAR proceed = false  # ✅ Escape route
  - "Cancel workflow" → RETURN  # ✅ Exit option
END OPTIONS
```

**Bad**:
```cheesecake
OPTIONS:
  - "Use Opus" → VAR model = "opus"
  - "Use Sonnet" → VAR model = "sonnet"
  # ❌ No way to cancel or skip
END OPTIONS
```

---

## Testing INTERACTIVE Blocks

### Mock Mode for Testing

When testing workflows with INTERACTIVE blocks:

```cheesecake
# Set test mode variable
VAR test_mode = true

IF test_mode:
  # In test mode, auto-select first option
  VAR auto_choice = "approve"
ELSE:
  # In normal mode, ask user
  INTERACTIVE AT "approval":
    ASK USER: "Approve?"
    OPTIONS:
      - "Approve" → VAR auto_choice = "approve"
      - "Reject" → VAR auto_choice = "reject"
    END OPTIONS
  END INTERACTIVE
END IF
```

### Test Workflow Example

```cheesecake
# tests/test-interactive.cheesecake

IMPORT "../src/workflow.cheesecake" AS workflow

# Mock INTERACTIVE blocks by pre-setting choices
VAR mock_choices = {
  "review-checkpoint": "approve",
  "cost-approval": "proceed",
  "final-review": "finalize"
}

# Run workflow with mocked interactions
VAR result = workflow.run_with_mocks(mock_choices)

ASSERT result.success == true
ASSERT result.output != NULL
```

---

## Summary

Interactive Mode enables:

✅ **Human-in-the-loop workflows** - User judgment guides AI
✅ **Decision points** - User chooses workflow path
✅ **Review checkpoints** - Approve before expensive ops
✅ **Cost control** - Zero cost during pauses
✅ **Iterative refinement** - User feedback mid-workflow

**Syntax**:
```cheesecake
INTERACTIVE AT "checkpoint-name":
  SHOW: {variable}
  ASK USER: "Question?"
  OPTIONS:
    - "option1" → action1
    - "option2" → action2
  END OPTIONS
END INTERACTIVE
```

**Rules**:
- Minimum 2 options, maximum 10
- Cannot nest INTERACTIVE blocks
- Cannot use inside PARALLEL blocks
- Zero cost during pause
- Works with AskUserQuestion tool

**Use Cases**:
- Approval workflows (cost, quality, safety)
- Iterative refinement with review
- Agent/model selection
- Path branching based on human judgment
- Early exit/cancel points

Interactive Mode makes CheeseCake workflows **collaborative** between AI and humans! 🤝
