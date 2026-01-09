# CheeseCake VM Execution Semantics
# Purpose: Define how AI agents execute .cheesecake programs
# Part of: CheeseCake v0.0.1 - Module 3 (VM Execution Engine)
# Updated: v0.0.2 - Added INTERACTIVE (Module 10), Cost Management (Module 11), Events & Scheduling (Module 12)
#
# This file explains the runtime behavior of the CheeseCake VM.
# You (the AI) ARE the VM when executing .cheesecake files.
#
# Usage:
#   Referenced by SKILL.md for execution behavior
#   Defines how to coordinate sessions, manage state, handle errors
#
# Dependencies:
#   - SKILL.md (language specification)
#   - Claude Code Task tool (for spawning sessions)
#
# Related:
#   - AGENT.md in agents/vm/ (VM agent definition)

---

# CheeseCake Virtual Machine Semantics

## Overview

When you execute a `.cheesecake` file, **you are the CheeseCake VM**. This document defines how you should behave as the interpreter and execution coordinator.

### Core Principles

1. **Session-as-Runtime**: The AI session executing the .cheesecake file IS the VM
2. **Sub-agent Spawning**: Use Claude Code's `Task` tool to create sessions for each agent
3. **Intelligent Coordination**: Apply AI understanding to coordinate execution
4. **State Management**: Track variables, checkpoints, and memory
5. **Progressive Execution**: Execute statements sequentially unless explicitly parallel

---

## 1. Execution Model Overview

### Execution Phases

```
┌─────────────────────────────────────────────────────┐
│  Phase 1: PARSE & VALIDATE                         │
│  - Read the .cheesecake file                       │
│  - Validate syntax against SKILL.md               │
│  - Build execution plan                            │
└──────────────┬──────────────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────────────┐
│  Phase 2: INITIALIZE                               │
│  - Create scope for variables                      │
│  - Load any existing checkpoints                   │
│  - Prepare state tracking                          │
└──────────────┬──────────────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────────────┐
│  Phase 3: EXECUTE                                  │
│  - Execute statements in order                     │
│  - Spawn sessions using Task tool                  │
│  - Coordinate parallel blocks                      │
│  - Evaluate semantic conditions                    │
│  - Update state                                    │
└──────────────┬──────────────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────────────┐
│  Phase 4: FINALIZE                                 │
│  - Save any pending checkpoints                    │
│  - Report results                                  │
│  - Cleanup resources                               │
└─────────────────────────────────────────────────────┘
```

---

## 2. Statement Execution Rules

### Sequential Execution (Default)

By default, statements execute **one after another**:

```cheesecake
VAR step1 = RUN SESSION(agent): TASK: "First"
VAR step2 = RUN SESSION(agent): TASK: "Second"
VAR step3 = RUN SESSION(agent): TASK: "Third"
```

**VM Behavior**:
1. Execute line 1 completely, wait for result
2. Execute line 2 completely, wait for result
3. Execute line 3 completely, wait for result

### Block Scoping

Variables declared in blocks have block scope unless they're updating outer scope variables:

```cheesecake
VAR outer = "initial"

IF **condition**:
  VAR inner = "block-local"  # Only exists in this block
  outer = "modified"          # Updates outer scope
END IF

# inner is not accessible here
# outer is "modified"
```

---

## 3. Agent & Session Management

### Creating Agent Instances

```cheesecake
VAR agent = NEW AgentName()
```

**VM Behavior**:
1. Look up the `AGENT AgentName` definition
2. If using `EXTENDS`, merge properties from parent
3. If using `IMPLEMENTS`, merge skills from all implemented skills
4. Create an in-memory representation of the agent configuration
5. Store in variable `agent`

### Creating Sessions

```cheesecake
VAR session = SESSION(agent):
  TASK: "Do something"
  INPUT: {data}
```

**VM Behavior**:
1. Take the agent configuration from `agent`
2. Build a session object with:
   - Model: from agent definition
   - Skills: from agent definition
   - System prompt: from agent definition
   - Task: the natural language task
   - Input: the provided data
   - Context: additional context (if provided)
3. Store session object in variable (not yet executed)

### Running Sessions

```cheesecake
VAR result = RUN session
```

**VM Behavior**:
1. Use Claude Code's `Task` tool to spawn a new session
2. Configure the session with:
   ```
   model: session.model
   prompt: session.system_prompt + "\n\nTask: " + session.task
   context: session.input + session.context
   ```
3. **Wait** for the session to complete
4. Capture the output
5. Store output in `result` variable

### Inline Session Creation & Running

```cheesecake
VAR result = RUN SESSION(agent):
  TASK: "Do something"
```

**VM Behavior**:
Same as creating session + running it, but done in one step.

---

## 4. Parallel Execution Coordination

### Basic Parallel Block

```cheesecake
PARALLEL:
  VAR r1 = RUN SESSION(agent1): TASK: "Task 1"
  VAR r2 = RUN SESSION(agent2): TASK: "Task 2"
  VAR r3 = RUN SESSION(agent3): TASK: "Task 3"
END PARALLEL
```

**VM Behavior**:
1. Identify all `RUN` statements in the parallel block
2. Spawn **all sessions concurrently** using multiple Task tool calls in a single message
3. **Wait for ALL to complete** (default join strategy)
4. Assign results to variables
5. Continue execution after the block

**Critical**: Use Claude Code's ability to call multiple tools in parallel. Send a single message with multiple `Task` tool invocations.

### Join Strategies

#### JOIN(first) - Race Mode

```cheesecake
PARALLEL JOIN(first):
  VAR fast = RUN SESSION(agent): TASK: "Fast task"
  VAR slow = RUN SESSION(agent): TASK: "Slow task"
END PARALLEL
```

**VM Behavior**:
1. Spawn all sessions concurrently
2. Wait for **first one** to complete
3. Cancel remaining sessions (if possible)
4. Use the first result for all variables in the block
5. Continue execution

#### JOIN(any, N) - Wait for N

```cheesecake
PARALLEL JOIN(any, 2):
  VAR r1 = RUN SESSION(agent): TASK: "Option 1"
  VAR r2 = RUN SESSION(agent): TASK: "Option 2"
  VAR r3 = RUN SESSION(agent): TASK: "Option 3"
END PARALLEL
```

**VM Behavior**:
1. Spawn all sessions concurrently
2. Wait for **any N** (here 2) to complete
3. Use completed results
4. Cancel remaining if desired
5. Continue execution

### Parallel For-Each

```cheesecake
PARALLEL FOR item IN items:
  VAR result = RUN SESSION(processor): TASK: "Process {item}"
END PARALLEL FOR
```

**VM Behavior**:
1. Create a session for each item in `items`
2. Spawn all sessions concurrently
3. Wait for all to complete
4. Store results in an array (in same order as input)
5. Continue execution

---

## 5. Semantic Condition Evaluation

Conditions in `**...**` are evaluated using AI understanding, not boolean logic.

### Example Conditions

```cheesecake
IF **{draft} is ready for publication**:
  # How to evaluate:
  # - Read the draft content
  # - Apply understanding of "publication ready":
  #   * Well-structured
  #   * No obvious errors
  #   * Coherent and complete
  # - Return true or false based on understanding
```

```cheesecake
LOOP UNTIL **{findings} shows conclusive evidence** MAX 5:
  # How to evaluate:
  # - Examine findings
  # - Determine if evidence is conclusive
  # - If yes, exit loop
  # - If no, continue
```

### Evaluation Guidelines

1. **Read the context**: Examine all referenced variables
2. **Apply understanding**: Use AI intelligence to interpret the condition
3. **Be conservative**: When uncertain, lean toward "false" to avoid premature exits
4. **Consider intent**: What is the user trying to achieve?
5. **Check progress**: If condition never changes, stop after MAX iterations

---

## 6. State Management

### Variables

**Tracking**:
- Maintain a variable table: `{ name → value }`
- Update on assignment
- Check existence before use
- Respect scope rules (block-local vs outer)

**Example State**:
```
Variables:
  agent: <Researcher instance config>
  findings: "Research output from session..."
  draft: "Article text..."
  iteration: 3
```

### Checkpoints

```cheesecake
CHECKPOINT "name":
  SAVE: {var1, var2}
  TO: ".cheesecake/state/"
END CHECKPOINT
```

**VM Behavior**:
1. Create directory `.cheesecake/state/` if not exists
2. Serialize variables `{var1, var2}` to JSON
3. Save to file `.cheesecake/state/name.json`
4. Include metadata:
   ```json
   {
     "checkpoint": "name",
     "timestamp": "ISO8601 timestamp",
     "variables": {
       "var1": ...,
       "var2": ...
     }
   }
   ```

### Restoring Checkpoints

```cheesecake
IF CHECKPOINT_EXISTS("name"):
  RESTORE FROM "name"
```

**VM Behavior**:
1. Check if `.cheesecake/state/name.json` exists
2. If yes, return true for CHECKPOINT_EXISTS
3. On RESTORE:
   - Load the JSON file
   - Deserialize variables
   - Update variable table with restored values

### Memory

```cheesecake
MEMORY project_state:
  phase: "research"
  history: []
END MEMORY
```

**VM Behavior**:
1. Create/update file `.cheesecake/memory/project_state.json`
2. Store the object
3. Persist across executions (different from checkpoints - these survive program end)

```cheesecake
MEMORY project_state.history.APPEND(new_item)
```

**VM Behavior**:
1. Load `.cheesecake/memory/project_state.json`
2. Navigate to `history` array
3. Append `new_item`
4. Save updated memory

---

## 7. Loop Execution

### Fixed Loops

```cheesecake
REPEAT 3:
  VAR result = RUN SESSION(agent): TASK: "Do something"
END REPEAT
```

**VM Behavior**:
1. Execute block 3 times
2. Variables in block can be updated each iteration
3. Continue after loop completes

### Semantic Loops

```cheesecake
LOOP UNTIL **{draft} is ready** MAX 5:
  VAR draft = RUN SESSION(writer): TASK: "Improve {draft}"
END LOOP
```

**VM Behavior**:
1. iteration = 0
2. While iteration < MAX:
   - Execute loop body
   - Evaluate semantic condition
   - If condition is true, exit loop
   - iteration++
3. If MAX reached, exit loop (safety)

**Critical**: Always respect MAX limit to prevent infinite loops.

---

## 8. Interactive Mode Execution (v0.0.2+)

### Overview

INTERACTIVE blocks pause workflow execution to request user input. The VM must:
1. **Save state** before pausing
2. **Present context** to the user
3. **Wait for user input** using AskUserQuestion tool
4. **Execute selected action**
5. **Resume execution** seamlessly

### Execution Flow

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

**VM Behavior**:

#### Step 1: Pre-Pause State Preservation

Before pausing, save current state:
```javascript
interactive_state = {
  checkpoint_name: "checkpoint-name",
  line_number: current_line,
  variables: deep_copy(all_variables),
  context: execution_context,
  timestamp: NOW()
}
```

#### Step 2: Display Context (if SHOW present)

If SHOW clause exists:
1. Evaluate the expression: `result = evaluate(variable_expression)`
2. Format for display:
   - If string: show directly
   - If object: pretty-print as JSON or summary
   - If array: show length + sample items
   - If long (>500 chars): show truncated + "see full output" link
3. Display to user before question

**Example**:
```cheesecake
SHOW: {draft}
# VM displays: "Current draft (2,340 words): \n[truncated text]..."

SHOW: {findings.summary}
# VM displays: "Quantum computing shows rapid advances..."

SHOW: "Estimated cost: ${total_cost}"
# VM displays: "Estimated cost: $0.45"
```

#### Step 3: Present Question and Options

Use `AskUserQuestion` tool:

```javascript
// Build options array for AskUserQuestion
options = []
for option in block.options:
  options.append({
    label: option.label,
    description: summarize_action(option.action)  // Brief description of what happens
  })

// Call AskUserQuestion
user_choice = AskUserQuestion({
  question: block.ask_user_text,
  header: block.checkpoint_name,
  options: options,
  multiSelect: false  // Interactive blocks are single-choice only
})
```

**Action Description Generation**:
- `VAR x = true` → "Set x to true"
- `BREAK` → "Exit current loop"
- `RETURN` → "End workflow execution"
- `RUN SESSION(...)` → "Execute [agent name] session"
- `CONTINUE` → "Continue to next iteration"

#### Step 4: Execute Selected Action

After user selects option:

1. Find the matching option by label: `selected = find_option(user_choice, block.options)`
2. Execute the action statement: `execute_statement(selected.action)`
3. Update variables as needed
4. Apply control flow effects (BREAK, CONTINUE, RETURN)

**Examples**:

```cheesecake
OPTIONS:
  - "approve" → VAR approved = true
  # User selects "approve"
  # VM executes: VAR approved = true
  # Result: approved = true

OPTIONS:
  - "finalize" → BREAK
  # User selects "finalize"
  # VM executes: BREAK
  # Result: Exit enclosing loop

OPTIONS:
  - "use opus" → VAR result = RUN SESSION(opus_agent): TASK: "..."
  # User selects "use opus"
  # VM executes: Spawns opus_agent session, waits for result, assigns to result
  # Result: result variable contains session output
```

#### Step 5: Resume Execution

After action completes:

1. Update progress tracking (if paused, now resume)
2. Continue with next statement after `END INTERACTIVE`
3. Execution proceeds normally

**State After Resume**:
- All variables preserved
- New variables from action available
- Control flow effects applied (if BREAK/CONTINUE/RETURN)
- Progress tracking resumes
- Time tracking resumes

### Special Cases

#### INTERACTIVE in Loops

When INTERACTIVE is inside a loop:

```cheesecake
LOOP UNTIL done MAX 5:
  VAR result = RUN SESSION(agent): TASK: "..."

  INTERACTIVE AT "review-iteration":
    SHOW: {result}
    ASK USER: "Continue?"
    OPTIONS:
      - "Yes" → CONTINUE
      - "No" → BREAK
    END OPTIONS
  END INTERACTIVE
END LOOP
```

**VM Behavior**:
- Each iteration can pause at INTERACTIVE
- User choice affects loop continuation (BREAK/CONTINUE)
- Loop state preserved during pause
- Iteration counter preserved

#### INTERACTIVE in Conditionals

```cheesecake
IF **{quality} is low**:
  INTERACTIVE AT "low-quality-handler":
    ASK USER: "Quality is low. What to do?"
    OPTIONS:
      - "Retry" → VAR retry = true
      - "Accept" → VAR retry = false
    END OPTIONS
  END INTERACTIVE
END IF
```

**VM Behavior**:
- INTERACTIVE only executes if condition is true
- Pause happens conditionally
- Variable updates from action available after block

#### Action Execution Failures

If the action statement fails during execution:

```cheesecake
OPTIONS:
  - "Option A" → VAR x = RUN SESSION(agent): TASK: "..."  # Could fail
END OPTIONS
```

**VM Behavior**:
1. Try to execute action
2. If fails (session error, timeout, etc.):
   - Log error: "Interactive action failed: [error]"
   - Apply normal error handling (try/catch if present)
   - Workflow may continue or abort based on error handling

### Progress Tracking Integration

When INTERACTIVE pauses execution:

**Progress Display**:
```
[■■■■■■□□□□] 60% complete

✓ Phase 1: Research           [DONE]     8.5s
⏸  Phase 2: Analysis          [PAUSED]   User input required
○ Phase 3: Writing            [PENDING]

[PAUSE] Waiting for user input at: approve-analysis
```

**Key Points**:
- Use ⏸ symbol for paused phase
- Show checkpoint name
- Clear indication of pause state
- Time counter pauses (not counted during user input)
- Token counter pauses (no new tokens during pause)

### Cost Tracking Integration

INTERACTIVE blocks have **zero cost**:

```cheesecake
# Cost estimate example
PHASE "Research":
  VAR findings = RUN SESSION(researcher): TASK: "..."  # $0.02
END PHASE

INTERACTIVE AT "review":
  SHOW: {findings}
  ASK USER: "Proceed?"
  OPTIONS:
    - "Yes" → VAR proceed = true
    - "No" → VAR proceed = false
  END OPTIONS
END INTERACTIVE  # $0.00 - no cost, no tokens

PHASE "Analysis":
  IF proceed:
    VAR analysis = RUN SESSION(analyst): TASK: "..."  # $0.05
  END IF
END PHASE
```

**VM Behavior for Cost Tracking**:
- INTERACTIVE blocks: 0 tokens, $0.00 cost
- Time pauses during user input (not counted)
- Cost only resumes after user provides input
- Display cost estimate ranges based on user choices

### Error Handling

#### User Doesn't Provide Input

If user session ends or times out before providing input:

**VM Behavior**:
1. Save checkpoint at INTERACTIVE point
2. Workflow status: **SUSPENDED**
3. Can resume later with `/cheesecake resume workflow.cheesecake`
4. Resume loads checkpoint and re-presents INTERACTIVE

#### Invalid Option Selected

Shouldn't happen (AskUserQuestion validates), but if it does:

**VM Behavior**:
1. Log warning: "Invalid option selected, using first option as fallback"
2. Execute first option's action
3. Continue execution

### Testing Interactive Workflows

When testing workflows with INTERACTIVE blocks:

**Test Mode Variable**:
```cheesecake
VAR test_mode = true  # Set by test runner

IF test_mode:
  # Mock: Auto-select first option without user input
  VAR auto_choice = "option1"
ELSE:
  # Normal: Ask user
  INTERACTIVE AT "checkpoint":
    ASK USER: "Choose?"
    OPTIONS:
      - "option1" → VAR auto_choice = "option1"
      - "option2" → VAR auto_choice = "option2"
    END OPTIONS
  END INTERACTIVE
END IF
```

### Constraints

**Not Allowed**:
- ❌ INTERACTIVE inside PARALLEL blocks (would break parallel semantics)
- ❌ Nested INTERACTIVE (INTERACTIVE inside INTERACTIVE)

**If encountered**:
- Parse Error: "INTERACTIVE cannot be used inside PARALLEL blocks"
- Parse Error: "Nested INTERACTIVE blocks are not allowed"

### Resume from Checkpoint

If workflow pauses at INTERACTIVE and user resumes later:

**Resume Process**:
1. Load checkpoint: `state = LOAD ".cheesecake/state/interactive-{checkpoint_name}.json"`
2. Restore variables: `restore_variables(state.variables)`
3. Restore execution context: `execution_context = state.context`
4. Re-present INTERACTIVE block (ask user again)
5. Execute selected action
6. Continue execution

**Example**:
```bash
# First run - pauses at INTERACTIVE
/cheesecake run workflow.cheesecake
# → Pauses, saves checkpoint, exits

# Later - resume from checkpoint
/cheesecake resume workflow.cheesecake
# → Loads checkpoint, re-presents INTERACTIVE, continues after user input
```

### Implementation Checklist for VM

When implementing INTERACTIVE execution:

- [ ] Save state before pause (variables, context, line number)
- [ ] Format and display SHOW expression if present
- [ ] Build options array with descriptions
- [ ] Call AskUserQuestion tool with correct parameters
- [ ] Parse user selection
- [ ] Execute action statement associated with selection
- [ ] Handle control flow effects (BREAK, CONTINUE, RETURN)
- [ ] Update progress tracking (pause/resume indicators)
- [ ] Ensure zero cost during pause
- [ ] Support resume from checkpoint
- [ ] Handle errors in action execution
- [ ] Validate constraints (no PARALLEL, no nesting)

---

## 9. Cost Management & Budget Tracking (v0.0.2+)

### Overview

Module 11 adds comprehensive cost management to CheeseCake workflows. The VM is responsible for:
- Tracking real-time costs during execution
- Checking operations against budget limits
- Displaying cost warnings to users
- Enforcing budget constraints
- Providing optimization suggestions

### CONFIG Block Parsing

When the VM encounters a CONFIG block:

```cheesecake
CONFIG:
  BUDGET: $1.00
  CONFIRM_COST_ABOVE: $0.10
  WARN_PARALLEL_ABOVE: 5
END CONFIG
```

**VM Actions**:
1. Parse CONFIG block (must be at start of file)
2. Validate all settings (dollar amounts, numbers, booleans)
3. Store configuration in execution context
4. Initialize cost tracking state:
   - `current_cost = $0.00`
   - `budget = CONFIG.BUDGET`
   - `warning_thresholds = {CONFIRM_COST_ABOVE, WARN_PARALLEL_ABOVE, etc.}`
5. Apply settings globally to workflow execution

**Default Values** (if CONFIG not present or setting omitted):
```javascript
{
  BUDGET: null,  // No limit
  CONFIRM_COST_ABOVE: null,  // No confirmations
  WARN_PARALLEL_ABOVE: null,  // No warnings
  WARN_AT_PERCENT: 90,
  OPTIMIZATION_SUGGESTIONS: true,
  DEFAULT_MODEL: "sonnet",
  MAX_PARALLEL_SESSIONS: null,  // Unlimited
  TIMEOUT_DEFAULT: "120s",
  STOP_ON_BUDGET_EXCEED: true,
  INTERACTIVE_WARNINGS: true
}
```

### Cost Tracking Protocol

**For every session execution**:

1. **Before session starts**:
   ```
   a) Estimate cost based on:
      - Model type (sonnet: ~$0.03, opus: ~$0.15, haiku: ~$0.001)
      - Task complexity (simple vs complex)
      - Context size
      - Estimated output length

   b) Check budget:
      - If current_cost + estimated_cost > CONFIG.BUDGET:
        → Trigger budget exceeded warning/error

   c) Check confirmation threshold:
      - If estimated_cost > CONFIG.CONFIRM_COST_ABOVE:
        → Show operation cost warning, wait for user input
   ```

2. **During session execution**:
   ```
   - Show estimated cost in progress display
   - Update progress with "(estimating...)" indicator
   ```

3. **After session completes**:
   ```
   a) Calculate actual cost:
      - actual_cost = (input_tokens × input_price) + (output_tokens × output_price)

   b) Update running total:
      - current_cost += actual_cost

   c) Update progress display with actual cost

   d) Check budget threshold:
      - If (current_cost / CONFIG.BUDGET) >= (CONFIG.WARN_AT_PERCENT / 100):
        → Show budget threshold warning (once)

   e) Log variance:
      - variance = actual_cost - estimated_cost
      - If |variance| > 20%: note for estimation improvement
   ```

### Cost Estimation Formulas

Use these formulas from Module 9 (cost-estimation.md):

**Base cost per model** (simple task, ~1000 tokens total):
- Sonnet: $0.003
- Opus: $0.015
- Haiku: $0.001

**Complexity multipliers**:
- Simple task: 1x
- Moderate task: 2x
- Complex task: 4x

**Context multipliers**:
- Small context (<1000 tokens): 1x
- Medium context (1000-5000 tokens): 1.5x
- Large context (>5000 tokens): 2.5x

**Formula**:
```
estimated_cost = base_cost × complexity_multiplier × context_multiplier × 1.2 (buffer)
```

### Budget Check Points

The VM checks budget at these points:

1. **Before each RUN SESSION**:
   - Check: `current_cost + estimated_session_cost <= CONFIG.BUDGET`
   - If fail: Trigger budget exceeded

2. **Before PARALLEL block**:
   - Check: `current_cost + sum(estimated_costs) <= CONFIG.BUDGET`
   - Count sessions: If count > CONFIG.WARN_PARALLEL_ABOVE, warn
   - If budget fail: Trigger budget exceeded

3. **Before each LOOP iteration**:
   - Check: `current_cost + estimated_iteration_cost <= CONFIG.BUDGET`
   - If fail: Trigger budget exceeded, exit loop early

4. **At CONFIG.WARN_AT_PERCENT threshold**:
   - Once current_cost crosses percentage, show threshold warning
   - Example: If WARN_AT_PERCENT: 80 and BUDGET: $1.00, warn at $0.80

### Warning Display Protocol

#### Operation Cost Warning

**Trigger**: `estimated_cost > CONFIG.CONFIRM_COST_ABOVE`

**Display Format**:
```
⚠️  COST WARNING
This operation estimated at: ~${estimated_cost}
Model: {model_name}
Task: "{task_preview...}"
Current budget: ${current_cost} / ${CONFIG.BUDGET} ({percentage}%)
After operation: ~${current_cost + estimated_cost} / ${CONFIG.BUDGET} ({new_percentage}%)

Continue? [Y/n/e]
  Y - Continue with operation
  n - Skip operation (variable = NULL)
  e - Edit task to reduce cost
```

**VM Actions**:
- If `CONFIG.INTERACTIVE_WARNINGS: false`: Auto-continue, log warning only
- If `CONFIG.INTERACTIVE_WARNINGS: true`: Wait for user input
- On 'Y': Continue normally
- On 'n': Skip operation, set variable to NULL, continue workflow
- On 'e': Prompt user to modify task, re-estimate, show updated warning

#### Parallel Session Warning

**Trigger**: `session_count > CONFIG.WARN_PARALLEL_ABOVE`

**Display Format**:
```
⚠️  PARALLEL SESSION WARNING
About to spawn {count} parallel sessions
Model: {model} (x{count})
Estimated total cost: ~${total_estimated_cost}
Current budget: ${current_cost} / ${CONFIG.BUDGET} ({percentage}%)
After operation: ~${current_cost + total_estimated_cost} / ${CONFIG.BUDGET} ({new_percentage}%)

Session breakdown:
  • Session 1: {task_preview} (~${cost1})
  • Session 2: {task_preview} (~${cost2})
  • ... ({remaining} more)

Continue? [Y/n/r]
  Y - Spawn all {count} sessions
  n - Skip entire PARALLEL block
  r - Reduce (enter max count)
```

**VM Actions**:
- On 'Y': Continue with all sessions
- On 'n': Skip PARALLEL block, all variables in block = NULL
- On 'r': Prompt for max count (e.g., "5"), spawn only first N sessions

#### Budget Threshold Warning

**Trigger**: `current_cost >= (CONFIG.BUDGET × CONFIG.WARN_AT_PERCENT / 100)`

**Display Format**:
```
⚠️  BUDGET THRESHOLD WARNING
Current cost: ${current_cost} / ${CONFIG.BUDGET} ({percentage}% of budget used)
Estimated remaining operations: ~${estimated_remaining}
You may exceed budget soon.

Continue? [Y/n]
```

**VM Actions**:
- Show warning once when threshold crossed
- If 'n': Stop workflow gracefully
- If 'Y': Continue, don't show this warning again

#### Budget Exceeded Error

**Trigger**: `current_cost + estimated_cost > CONFIG.BUDGET` (when `STOP_ON_BUDGET_EXCEED: true`)

**Display Format**:
```
❌ BUDGET EXCEEDED
Current cost: ${current_cost}
Next operation estimated: ~${estimated_cost}
Total would be: ${current_cost + estimated_cost} (exceeds ${CONFIG.BUDGET} budget)

Operation: {operation_type} at line {line_number}
Task: "{task_preview...}"

Workflow stopped to prevent budget overrun.

Options:
1. Increase budget: Edit CONFIG: BUDGET: ${suggested_budget}
2. Use cheaper model: Switch {current_model} → {cheaper_model} (saves ~{savings})
3. Skip operation: Comment out or remove this task
4. Add checkpoint: Resume from here after budget adjustment

Run with --estimate to see full cost breakdown.
```

**VM Actions** (STOP_ON_BUDGET_EXCEED: true):
- Stop workflow immediately
- Do not execute operation
- Exit with error code 1
- Log final cost summary

**VM Actions** (STOP_ON_BUDGET_EXCEED: false):
- Show warning
- Allow workflow to continue
- Log overage
- Continue tracking (cost can exceed budget)

### Progress Display Integration

Cost information must appear in progress display:

**Format**:
```
┌─────────────────────────────────────────────────────┐
│  Running: workflow.cheesecake                       │
│                                                     │
│  [■■■■■■□□□□] 60% complete                         │
│                                                     │
│  ✓ Phase 1: Research          [DONE]    2.3s       │
│  → Phase 2: Analysis          [RUNNING] (est. $0.15)│
│  ○ Phase 3: Writing           [PENDING]            │
│                                                     │
│  Tokens: 12,450 used | ~8,000 remaining            │
│  Cost: $0.12 / $2.00 budget (6% used) ✓           │
└─────────────────────────────────────────────────────┘
```

**Cost Display Indicators**:
- ✓ (Green): Within budget, <75% used
- ⚠ (Yellow): Approaching budget, 75-100% used
- ❌ (Red): Budget exceeded, >100%

**Update frequency**:
- Update after each session completes
- Show "(estimating...)" during session execution
- Update percentage in real-time

### Optimization Suggestions

**When to generate suggestions**:
1. After workflow completes successfully
2. When budget threshold warning triggered
3. During dry-run mode

**Types of suggestions**:

1. **Model Downgrade**:
```
💡 OPTIMIZATION #1: Model Downgrade
Location: Line {line_number}
Current: {current_model} for task "{task}"
Cost: ~${cost} per iteration ({iterations} iterations) = ~${total}

Suggestion: Switch to {cheaper_model}
Potential savings: ~${savings} ({percentage}% reduction)
Quality impact: {impact_level}
```

2. **Parallelize Sequential Operations**:
```
💡 OPTIMIZATION #2: Parallelize
Location: Lines {start}-{end}
Current: Sequential execution ({count} tasks)
Time: ~{time} seconds

Suggestion: Use PARALLEL block
Potential savings: ~{time_saved} seconds ({percentage}% faster)
```

3. **Add Checkpoints**:
```
💡 OPTIMIZATION #3: Add Checkpoint
Location: After line {line_number}
Risk: If workflow fails, ${cost} of work is lost

Suggestion: Add CHECKPOINT "{name}"
Benefit: Resume from checkpoint, avoid re-running expensive operations
```

4. **Batch Instead of Loop**:
```
💡 OPTIMIZATION #4: Batch Processing
Location: Line {line_number}
Current: Loop with {iterations} iterations (~${cost})

Suggestion: Batch all items in single session
Potential savings: ~${savings} ({percentage}% reduction)
```

**Display Format** (at end of execution):
```
============================================
OPTIMIZATION SUGGESTIONS
============================================

You have {count} potential optimizations:

💡 #1: Model Downgrade (Line {line})
   Potential savings: ~${savings} ({percentage}%)

💡 #2: Parallelize (Lines {start}-{end})
   Potential time savings: ~{time} seconds

Total potential savings: ~${total_savings}

View details: /cheesecake explain {file} --suggestions
Apply suggestions: /cheesecake optimize {file}
============================================
```

### VM Implementation Checklist

For cost management to work, the VM must:

- [ ] Parse CONFIG block at start of execution
- [ ] Validate all CONFIG settings
- [ ] Initialize cost tracking state
- [ ] Estimate cost before each operation
- [ ] Check budget before each operation
- [ ] Track actual cost after each operation
- [ ] Update progress display with cost info
- [ ] Show warnings at appropriate thresholds
- [ ] Enforce budget limit (if STOP_ON_BUDGET_EXCEED: true)
- [ ] Generate optimization suggestions
- [ ] Display final cost summary
- [ ] Handle CONFIG.INTERACTIVE_WARNINGS: false mode
- [ ] Integrate with dry-run mode (zero actual cost)
- [ ] Persist cost data with checkpoints
- [ ] Calculate accurate cost based on token usage

### Integration with Other Features

**With Module 9 (Estimation)**:
- Use estimation formulas for cost prediction
- Dry-run shows estimated costs without tracking actual
- Estimate command shows costs without executing

**With Module 10 (Interactive Mode)**:
- Cost warnings can trigger INTERACTIVE pauses
- User can approve/deny expensive operations
- Zero cost during INTERACTIVE pause

**With Progress Tracking**:
- Cost displayed in progress visualization
- Phase-by-phase cost breakdown
- Real-time cost updates

**With Checkpoints**:
- Cost data saved with checkpoint
- Resume restores cost state
- Prevents re-counting costs after resume

---

## 10. Event & Schedule Execution (v0.0.2+)

This section defines how the VM handles events and schedules. See `events.md` for complete specification.

### Event Registration

During the PARSE phase, the VM collects all event handlers:

```
EventRegistry = {
  "file_changed": [
    {handler: handler1, where: "path MATCHES 'src/**'", params: ["path", "type"]},
    {handler: handler2, where: null, params: ["path"]}
  ],
  "custom_event": [
    {handler: handler3, where: "status == 'active'", params: ["data", "status"]}
  ]
}

ListenerRegistry = {
  "data_ready": [handler4, handler5],
  "task_complete": [handler6]
}

ScheduleRegistry = {
  "health_check": {
    type: "INTERVAL",
    value: "1h",
    task: taskBlock,
    retry: 2,
    onFailure: failureAction
  }
}
```

### Event Dispatch Protocol

When an event occurs (via EMIT or external trigger):

```
dispatch_event(event_name, event_data):
  1. Look up event_name in EventRegistry and ListenerRegistry
  2. For each matching handler:
     a. Create isolated handler scope
     b. Bind event parameters to variables
     c. If WHERE clause present:
        - Evaluate condition
        - Skip handler if false
     d. Execute handler body
     e. Catch any errors (log, don't propagate)
     f. Continue to next handler
  3. Track event chain depth (prevent infinite loops)
  4. Return (events don't return values)
```

### EMIT Execution

When VM encounters EMIT statement:

```cheesecake
EMIT task_complete(task_id: "123", status: "success")
```

**VM Behavior:**

```
1. Build event object: {task_id: "123", status: "success"}
2. Check event chain depth < MAX_EVENT_DEPTH (default: 10)
3. If depth exceeded:
   - LOG WARNING: "Event chain limit reached"
   - Return without dispatching
4. Increment event chain depth
5. Call dispatch_event("task_complete", event_object)
6. Decrement event chain depth
```

### ON EVENT Execution

When handler matches:

```cheesecake
ON EVENT file_changed(path, type) WHERE path MATCHES "*.ts":
  LOG INFO: "TS file changed: {path}"
  VAR result = RUN SESSION(linter): TASK: "Lint {path}"
END ON
```

**VM Behavior:**

```
1. Receive event: {path: "src/app.ts", type: "modified"}
2. Check WHERE clause: "src/app.ts" MATCHES "*.ts" → true
3. Create handler scope with:
   - path = "src/app.ts"
   - type = "modified"
4. Execute handler body:
   - LOG INFO: "TS file changed: src/app.ts"
   - VAR result = RUN SESSION(linter)...
5. Cleanup handler scope
6. If error occurred, log but continue
```

### LISTEN FOR Execution

Lightweight event listener:

```cheesecake
LISTEN FOR data_ready:
  PRINT event.source
END LISTEN
```

**VM Behavior:**

```
1. Receive event: {source: "api", items: [...]}
2. Create handler scope with:
   - event = {source: "api", items: [...]}
3. Execute handler body using event.* notation
4. Cleanup handler scope
```

### Schedule Registration

Schedules are registered but not auto-executed (requires runtime):

```cheesecake
SCHEDULE hourly_check:
  INTERVAL: 1h
  TASK: RUN SESSION(checker): TASK: "Check"
  RETRY: 2
END SCHEDULE
```

**VM Behavior:**

```
1. Validate schedule:
   - Exactly one timing specifier (INTERVAL, CRON, ONCE_AT)
   - TASK is present
   - Duration format valid
2. Register in ScheduleRegistry:
   {
     name: "hourly_check",
     type: "INTERVAL",
     value: "1h",
     task: <task block>,
     retry: 2,
     startAt: null,
     endAt: null
   }
3. Log: "Schedule 'hourly_check' registered (INTERVAL: 1h)"
```

### Manual Schedule Trigger

For testing, schedules can be triggered manually:

```
/cheesecake trigger hourly_check
```

**VM Behavior:**

```
1. Look up "hourly_check" in ScheduleRegistry
2. If not found: ERROR "Unknown schedule"
3. Execute task block:
   - Track retry attempts
   - On failure:
     - If retries remaining, retry
     - Else execute ON_FAILURE
   - On success: execute ON_SUCCESS
4. Report result
```

### Event Filtering (WHERE Clause)

**Literal Comparisons:**
```
path == "src/app.ts"        → exact match
status != "pending"         → not equal
count > 10                  → numeric comparison
```

**Pattern Matching:**
```
path MATCHES "*.ts"         → glob pattern
path MATCHES "src/**/*.js"  → recursive glob
```

**Semantic Conditions:**
```
**{payload} contains error data**  → AI evaluation
**{issue} is urgent**              → AI understanding
```

**Combined:**
```
path MATCHES "src/**" AND type == "modified" AND **{path} is not test file**
```

### Event Chain Prevention

To prevent infinite event loops:

```
MAX_EVENT_DEPTH = 10

EventChainTracker:
  depth: 0
  chain: []

before_dispatch(event):
  if depth >= MAX_EVENT_DEPTH:
    LOG WARNING: "Event chain limit reached"
    return SKIP
  depth++
  chain.push(event_name)

after_dispatch():
  depth--
  chain.pop()
```

### Handler Error Isolation

Errors in one handler don't affect others:

```cheesecake
ON EVENT my_event():
  THROW "Error in handler 1"  # Logged, continues
END ON

ON EVENT my_event():
  LOG INFO: "Handler 2"       # Still executes
END ON
```

**VM Behavior:**

```
dispatch "my_event":
  handler 1:
    - Attempt execution
    - THROW caught
    - LOG ERROR: "Handler error: Error in handler 1"
    - Continue to next handler
  handler 2:
    - Execute normally
    - LOG INFO: "Handler 2"
```

### Schedule Task Execution

When a schedule fires (manually or via daemon):

```cheesecake
SCHEDULE backup:
  INTERVAL: 1d
  TASK:
    VAR result = RUN SESSION(backup_agent): TASK: "Backup"
    SAVE result TO "backup/latest.json"
  END TASK
  RETRY: 3
  ON_FAILURE: EMIT backup_failed()
  ON_SUCCESS: LOG SUCCESS: "Backup complete"
END SCHEDULE
```

**VM Behavior:**

```
execute_schedule("backup"):
  attempts = 0
  max_attempts = 3 + 1  # RETRY + initial

  while attempts < max_attempts:
    try:
      execute task block
      execute ON_SUCCESS (if present)
      return SUCCESS
    catch error:
      attempts++
      if attempts < max_attempts:
        LOG WARNING: "Retry {attempts}/{RETRY}"
      else:
        execute ON_FAILURE (if present)
        return FAILURE
```

### Integration with Other Features

**With Progress Tracking:**
- Event handlers show in progress display
- Handler execution time tracked
- Event dispatch shown as progress update

**With Cost Management:**
- Sessions in event handlers count toward budget
- Warnings apply to handler sessions
- Cost tracked per handler

**With INTERACTIVE:**
- Events can trigger INTERACTIVE blocks
- EMIT can be used in INTERACTIVE options

**With Checkpoints:**
- Event/schedule registrations saved with checkpoint
- On resume, handlers are re-registered

### Progress Display During Events

```
🎯 Executing: workflow.cheesecake

→ Event dispatched: file_changed
  ⏳ Handler 1: Linting src/app.ts...
  ✓ Handler 1: Complete (1.2s)
  ⏳ Handler 2: Logging...
  ✓ Handler 2: Complete (0.1s)
```

### VM Implementation Checklist

For VM implementers:

- [ ] Parse and collect ON EVENT declarations
- [ ] Parse and collect LISTEN FOR declarations
- [ ] Parse and collect SCHEDULE declarations
- [ ] Build EventRegistry, ListenerRegistry, ScheduleRegistry
- [ ] Implement dispatch_event function
- [ ] Implement EMIT execution
- [ ] Implement WHERE clause evaluation
- [ ] Implement MATCHES pattern matching
- [ ] Track event chain depth
- [ ] Isolate handler errors
- [ ] Support manual schedule triggers
- [ ] Integrate with cost tracking
- [ ] Update progress display for events

---

## 11. Error Handling

### Try/Catch

```cheesecake
TRY:
  VAR result = RUN SESSION(agent): TASK: "Risky operation"
CATCH error:
  LOG ERROR: "Failed: {error.message}"
  VAR result = fallback
FINALLY:
  CLEANUP resources
END TRY
```

**VM Behavior**:
1. Execute TRY block
2. If any statement raises an error:
   - Capture error details (message, type, context)
   - Jump to CATCH block
   - Provide error object to CATCH
   - Execute CATCH block
3. Always execute FINALLY block (if present)
4. Continue after TRY/CATCH/FINALLY

### Retry Logic

```cheesecake
VAR data = RUN SESSION(agent):
  TASK: "Fetch data"
  RETRY: 3
  BACKOFF: exponential
```

**VM Behavior**:
1. attempt = 0
2. While attempt < RETRY:
   - Try to run session
   - If succeeds, return result
   - If fails:
     - Calculate backoff delay:
       - none: 0s
       - linear: attempt * 1s
       - exponential: 2^attempt seconds
     - Wait delay
     - attempt++
3. If all retries fail, raise error

---

## 12. Context Passing

### Between Sessions

```cheesecake
VAR findings = RUN SESSION(researcher): TASK: "Research AI"

VAR article = RUN SESSION(writer):
  TASK: "Write article about AI trends"
  INPUT: {findings}
```

**VM Behavior**:
1. When running `writer` session, include `findings` content in the context
2. Format as:
   ```
   System: You are a creative writer...

   Task: Write article about AI trends

   Input Data:
   findings: <content of findings variable>
   ```
3. Let the agent access the input naturally

---

## 13. Function Calls

### Execution

```cheesecake
FUNCTION research_topic(topic):
  VAR agent = NEW Researcher()
  VAR result = RUN SESSION(agent): TASK: "Research {topic}"
  RETURN result
END FUNCTION

VAR output = CALL research_topic(topic: "quantum computing")
```

**VM Behavior**:
1. On CALL:
   - Look up function definition
   - Create new scope for function execution
   - Bind arguments to parameters: `topic = "quantum computing"`
   - Execute function body in new scope
   - Capture RETURN value
   - Restore previous scope
   - Assign return value to `output`

---

## 14. Import/Export

### Import

```cheesecake
IMPORT "common/agents.cheesecake" AS lib
VAR agent = NEW lib.Researcher()
```

**VM Behavior**:
1. Read `common/agents.cheesecake`
2. Execute it in isolated scope
3. Capture all EXPORT-ed definitions
4. Store under namespace `lib`
5. Access via `lib.Name`

---

## 15. Progress Reporting

### Reporting Format

During execution, report progress to user:

```
┌─────────────────────────────────────────────────────┐
│  Running: workflow.cheesecake                       │
│                                                     │
│  [■■■□□□] 50% complete                             │
│                                                     │
│  ✓ Defined agents                                  │
│  ✓ Phase 1: Research (parallel)    [DONE]         │
│  → Phase 2: Writing                [RUNNING]       │
│  ○ Phase 3: Output                 [PENDING]       │
└─────────────────────────────────────────────────────┘
```

### Markers

- `✓` Complete
- `→` Currently running
- `○` Pending
- `⚠️` Warning
- `❌` Error

---

## 16. Special Behaviors

### Semantic Inference

When variables aren't explicitly declared but are clearly needed:

```cheesecake
REPEAT 5 AS index:
  PRINT "Iteration {index}"
```

**VM Behavior**: Automatically create and manage `index` variable within loop scope.

### Auto-summarization

If a variable contains very large content (>10k tokens), automatically summarize when passing to next session to avoid context overflow.

### Smart Context

When passing multiple variables as INPUT, intelligently format them for readability.

---

## 17. Error Messages

Provide clear, actionable error messages:

**Good**:
```
ERROR at line 15: Agent 'researcher' not defined

Suggestions:
1. Did you mean 'Researcher'? (case-sensitive)
2. Define the agent before use:
   AGENT researcher:
     MODEL: sonnet
```

**Bad**:
```
Error: undefined variable
```

---

## 18. Complete Execution Example

### Input File: `workflow.cheesecake`

```cheesecake
AGENT Researcher:
  MODEL: sonnet
  PROMPT: "You are a researcher."

VAR r = NEW Researcher()

PARALLEL:
  VAR f1 = RUN SESSION(r): TASK: "Research topic A"
  VAR f2 = RUN SESSION(r): TASK: "Research topic B"
END PARALLEL

IF **{f1} and {f2} are both comprehensive**:
  PRINT "Research complete!"
END IF
```

### Execution Trace

```
Step 1: Parse & Validate
  - Syntax valid ✓
  - Agent defined ✓
  - Variables declared properly ✓

Step 2: Initialize
  - Variable table: {}
  - Scope: global
  - State: clean

Step 3: Execute line-by-line

  Line 1-3: AGENT definition
    - Register Researcher agent config
    - Model: sonnet
    - Prompt: "You are a researcher."

  Line 5: VAR r = NEW Researcher()
    - Create agent instance
    - Store in variable table: r → <Researcher config>

  Line 7-10: PARALLEL block
    - Spawn 2 sessions concurrently
    - Session 1 task: "Research topic A"
    - Session 2 task: "Research topic B"
    - Wait for both to complete
    - Results:
      f1 → "Findings about topic A..."
      f2 → "Findings about topic B..."

  Line 12-14: IF statement
    - Evaluate condition: "both comprehensive"
    - Check f1 content: appears comprehensive ✓
    - Check f2 content: appears comprehensive ✓
    - Condition: TRUE
    - Execute IF block
    - Output: "Research complete!"

Step 4: Finalize
  - No checkpoints to save
  - Cleanup complete
  - Report: SUCCESS

Final state:
  Variables:
    r: <Researcher config>
    f1: "Findings about topic A..."
    f2: "Findings about topic B..."
```

---

## Summary

As the CheeseCake VM:

1. **Parse** .cheesecake files using SKILL.md
2. **Execute** statements sequentially or in parallel as specified
3. **Spawn** sub-agent sessions using Claude Code Task tool
4. **Evaluate** semantic conditions using AI understanding
5. **Manage** variables, checkpoints, and memory
6. **Report** progress and results
7. **Handle** errors gracefully

You are an intelligent interpreter, not a dumb parser. Use understanding to execute the user's intent.
