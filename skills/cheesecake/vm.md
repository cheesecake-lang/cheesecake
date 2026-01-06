# CheeseCake VM Execution Semantics
# Purpose: Define how AI agents execute .cheesecake programs
# Part of: CheeseCake v0.0.1 - Module 3 (VM Execution Engine)
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

## 8. Error Handling

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

## 9. Context Passing

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

## 10. Function Calls

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

## 11. Import/Export

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

## 12. Progress Reporting

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

## 13. Special Behaviors

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

## 14. Error Messages

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

## 15. Complete Execution Example

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
