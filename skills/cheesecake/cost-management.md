# CheeseCake Cost Management Specification
# Version: 0.0.2 (Module 11)
# Purpose: Complete specification for cost tracking, budgeting, and optimization

---

## Table of Contents

1. [Overview](#overview)
2. [CONFIG Block](#config-block)
3. [Cost Tracking](#cost-tracking)
4. [Warning System](#warning-system)
5. [Budget Enforcement](#budget-enforcement)
6. [Optimization Suggestions](#optimization-suggestions)
7. [Integration with Estimation](#integration-with-estimation)
8. [Examples](#examples)
9. [Best Practices](#best-practices)
10. [Testing Guidelines](#testing-guidelines)

---

## Overview

### Purpose

Cost Management (Module 11) provides comprehensive tools for controlling and optimizing the cost of AI agent workflows:

- **Prevent cost overruns**: Set hard budget limits
- **Informed decisions**: Get warnings before expensive operations
- **Optimize workflows**: Receive suggestions for cost reduction
- **Track spending**: Monitor real-time cost during execution
- **Budget planning**: Integrate with cost estimation (Module 9)

### Key Concepts

1. **CONFIG Block**: Global configuration for cost policies
2. **Cost Tracking**: Real-time cost accumulation during execution
3. **Warnings**: User prompts before expensive operations
4. **Budget Enforcement**: Automatic workflow stops when budget exceeded
5. **Optimization**: AI-suggested improvements for cost reduction

### When to Use

Use cost management when:
- Running workflows in production with cost constraints
- Experimenting with expensive models (Opus)
- Processing large datasets with many sessions
- Using PARALLEL blocks that spawn many agents
- Iterating in LOOP constructs with uncertain exit conditions

---

## CONFIG Block

### Syntax

```cheesecake
CONFIG:
  # Cost Management Settings
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

### Settings Reference

#### Cost Management Settings

**BUDGET** (optional)
- **Type**: Dollar amount (e.g., `$1.00`, `$0.50`)
- **Default**: No limit
- **Purpose**: Maximum total cost for entire workflow
- **Behavior**: When exceeded, workflow stops (if STOP_ON_BUDGET_EXCEED: true)

```cheesecake
CONFIG:
  BUDGET: $1.00  # Workflow stops if cost exceeds $1.00
END CONFIG
```

**CONFIRM_COST_ABOVE** (optional)
- **Type**: Dollar amount (e.g., `$0.10`)
- **Default**: No confirmations
- **Purpose**: Ask user before operations exceeding this cost
- **Behavior**: Shows warning with estimated cost, waits for Y/n

```cheesecake
CONFIG:
  CONFIRM_COST_ABOVE: $0.10  # Confirm operations > $0.10
END CONFIG
```

**WARN_PARALLEL_ABOVE** (optional)
- **Type**: Integer (e.g., `5`, `10`)
- **Default**: No warnings
- **Purpose**: Warn when PARALLEL block spawns N+ sessions
- **Behavior**: Shows warning with session count and cost estimate

```cheesecake
CONFIG:
  WARN_PARALLEL_ABOVE: 5  # Warn if spawning 5+ parallel sessions
END CONFIG
```

**WARN_AT_PERCENT** (optional)
- **Type**: Integer 1-100 (e.g., `80`)
- **Default**: 90
- **Purpose**: Warn when budget usage reaches percentage
- **Behavior**: Shows warning once when threshold crossed

```cheesecake
CONFIG:
  BUDGET: $1.00
  WARN_AT_PERCENT: 80  # Warn at $0.80
END CONFIG
```

**OPTIMIZATION_SUGGESTIONS** (optional)
- **Type**: Boolean (`true` or `false`)
- **Default**: true
- **Purpose**: Enable/disable optimization suggestions
- **Behavior**: Shows suggestions after operations or at end

```cheesecake
CONFIG:
  OPTIMIZATION_SUGGESTIONS: false  # Disable suggestions
END CONFIG
```

#### Model Settings

**DEFAULT_MODEL** (optional)
- **Type**: Model name (`sonnet`, `opus`, `haiku`)
- **Default**: sonnet
- **Purpose**: Default model when agent doesn't specify
- **Behavior**: Used for any agent without MODEL property

```cheesecake
CONFIG:
  DEFAULT_MODEL: haiku  # Use Haiku by default
END CONFIG
```

#### Execution Settings

**MAX_PARALLEL_SESSIONS** (optional)
- **Type**: Integer (e.g., `10`, `20`)
- **Default**: Unlimited
- **Purpose**: Hard limit on concurrent sessions
- **Behavior**: PARALLEL blocks limited to this count

```cheesecake
CONFIG:
  MAX_PARALLEL_SESSIONS: 10  # Max 10 concurrent sessions
END CONFIG
```

**TIMEOUT_DEFAULT** (optional)
- **Type**: Time duration (e.g., `30s`, `2m`, `5m`)
- **Default**: 120s (2 minutes)
- **Purpose**: Default timeout for sessions without TIMEOUT
- **Behavior**: Sessions timeout after this duration

```cheesecake
CONFIG:
  TIMEOUT_DEFAULT: 30s  # 30 second default timeout
END CONFIG
```

#### Behavior Settings

**STOP_ON_BUDGET_EXCEED** (optional)
- **Type**: Boolean (`true` or `false`)
- **Default**: true
- **Purpose**: Whether to stop workflow when budget exceeded
- **Behavior**: If false, warns but continues

```cheesecake
CONFIG:
  BUDGET: $1.00
  STOP_ON_BUDGET_EXCEED: false  # Warn but don't stop
END CONFIG
```

**INTERACTIVE_WARNINGS** (optional)
- **Type**: Boolean (`true` or `false`)
- **Default**: true
- **Purpose**: Whether warnings require user input
- **Behavior**: If false, warnings are logged but auto-continue

```cheesecake
CONFIG:
  CONFIRM_COST_ABOVE: $0.10
  INTERACTIVE_WARNINGS: false  # Log warnings, auto-continue
END CONFIG
```

---

### CONFIG Block Rules

1. **Location**: Must appear at the start of the file (before any code)
2. **Uniqueness**: Only one CONFIG block per workflow
3. **Scope**: Applies to entire workflow execution
4. **Optional**: CONFIG block is completely optional
5. **Defaults**: All settings have sensible defaults
6. **Override**: Individual sessions can override TIMEOUT

### Examples

#### Minimal CONFIG
```cheesecake
CONFIG:
  BUDGET: $1.00
END CONFIG
```

#### Comprehensive CONFIG
```cheesecake
CONFIG:
  # Cost controls
  BUDGET: $2.00
  CONFIRM_COST_ABOVE: $0.15
  WARN_PARALLEL_ABOVE: 5
  WARN_AT_PERCENT: 75

  # Execution
  DEFAULT_MODEL: sonnet
  MAX_PARALLEL_SESSIONS: 10
  TIMEOUT_DEFAULT: 60s

  # Behavior
  STOP_ON_BUDGET_EXCEED: true
  INTERACTIVE_WARNINGS: true
  OPTIMIZATION_SUGGESTIONS: true
END CONFIG
```

#### Production CONFIG (Conservative)
```cheesecake
CONFIG:
  BUDGET: $0.50
  CONFIRM_COST_ABOVE: $0.05
  WARN_PARALLEL_ABOVE: 3
  DEFAULT_MODEL: sonnet
  MAX_PARALLEL_SESSIONS: 5
  STOP_ON_BUDGET_EXCEED: true
END CONFIG
```

#### Development CONFIG (Permissive)
```cheesecake
CONFIG:
  BUDGET: $5.00
  CONFIRM_COST_ABOVE: $0.50
  DEFAULT_MODEL: opus
  STOP_ON_BUDGET_EXCEED: false
  OPTIMIZATION_SUGGESTIONS: true
END CONFIG
```

---

## Cost Tracking

### How Costs Are Tracked

The VM tracks costs in real-time during workflow execution:

1. **Before each session**: Estimate cost based on model and task
2. **After each session**: Record actual tokens used
3. **Accumulate total**: Add to running total
4. **Update display**: Show in progress visualization
5. **Check budget**: Compare against CONFIG.BUDGET

### Cost Calculation

**Per-session cost** = (Input tokens × Input price) + (Output tokens × Output price)

**Model pricing** (as of 2025):
- **Sonnet 4.5**: $3 / $15 per million tokens (input / output)
- **Opus 4.5**: $15 / $75 per million tokens (input / output)
- **Haiku 3.5**: $0.80 / $4 per million tokens (input / output)

### Cost Estimation

Before executing an operation, the VM estimates cost:

```
Estimated cost = Base cost + (Context size × price) + (Estimated output × price)
```

**Base costs** (typical):
- Simple task (Sonnet): ~$0.003
- Simple task (Opus): ~$0.015
- Simple task (Haiku): ~$0.001
- Complex task (Sonnet): ~$0.01
- Complex task (Opus): ~$0.05
- Complex task (Haiku): ~$0.002

**Task complexity factors**:
- Simple: Short prompt, minimal context, short output
- Complex: Long prompt, large context, detailed output

### Progress Display Integration

Cost information appears in progress display:

```
┌─────────────────────────────────────────────────────┐
│  Running: workflow.cheesecake                       │
│                                                     │
│  [■■■■■■□□□□] 60% complete                         │
│                                                     │
│  ✓ Phase 1: Research          [DONE]    2.3s       │
│  → Phase 2: Analysis          [RUNNING]            │
│  ○ Phase 3: Writing           [PENDING]            │
│                                                     │
│  Tokens: 12,450 used | ~8,000 remaining            │
│  Cost: $0.12 / $2.00 budget (6% used) ✓           │
└─────────────────────────────────────────────────────┘
```

### Cost Display Indicators

- **✓** (Green): Within budget, comfortable margin
- **⚠** (Yellow): Approaching budget (>75%)
- **❌** (Red): Budget exceeded

### Cost Tracking in Dry-Run

In dry-run mode (`--dry-run`):
- No actual costs incurred (zero cost)
- Shows estimated costs only
- Cost tracking simulated based on estimates
- Useful for planning and budgeting

---

## Warning System

### Warning Types

The VM can issue four types of cost warnings:

1. **Operation Cost Warning**: Single operation exceeds threshold
2. **Parallel Session Warning**: Many parallel sessions about to spawn
3. **Budget Threshold Warning**: Budget usage reaches percentage
4. **Budget Exceeded Error**: Budget limit exceeded

### 1. Operation Cost Warning

**Trigger**: Single operation estimated above `CONFIG.CONFIRM_COST_ABOVE`

**Format**:
```
⚠️  COST WARNING
This operation estimated at: ~$0.15
Model: opus
Task: "Comprehensive market analysis across 50 companies"
Current budget: $0.05 / $1.00 (5%)
After operation: ~$0.20 / $1.00 (20%)
Continue? [Y/n]
```

**User options**:
- `Y` (Yes): Continue with operation
- `n` (No): Skip operation, variable set to NULL
- `e` (Edit): Modify task to reduce cost

**Example**:
```cheesecake
CONFIG:
  CONFIRM_COST_ABOVE: $0.10
END CONFIG

AGENT OpusAgent:
  MODEL: opus
  PROMPT: "High-quality agent"

VAR agent = NEW OpusAgent()

# This triggers warning (~$0.15)
VAR result = RUN SESSION(agent):
  TASK: "Write comprehensive 2000-word analysis"
```

### 2. Parallel Session Warning

**Trigger**: PARALLEL block spawns more than `CONFIG.WARN_PARALLEL_ABOVE` sessions

**Format**:
```
⚠️  PARALLEL SESSION WARNING
About to spawn 10 parallel sessions
Model: sonnet (x10)
Estimated total cost: ~$0.30
Current budget: $0.10 / $1.00 (10%)
After operation: ~$0.40 / $1.00 (40%)

Session breakdown:
  • RUN SESSION(researcher): "Task 1" (~$0.03)
  • RUN SESSION(researcher): "Task 2" (~$0.03)
  • ... (8 more)

Continue? [Y/n/r]
```

**User options**:
- `Y` (Yes): Spawn all parallel sessions
- `n` (No): Skip entire PARALLEL block
- `r` (Reduce): Ask for max count (e.g., "5" → only spawn first 5)

**Example**:
```cheesecake
CONFIG:
  WARN_PARALLEL_ABOVE: 5
END CONFIG

AGENT Worker:
  MODEL: sonnet
  PROMPT: "Worker"

VAR items = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# This triggers warning (10 sessions)
PARALLEL FOR item IN items:
  VAR result = RUN SESSION(worker):
    TASK: "Process {item}"
END PARALLEL FOR
```

### 3. Budget Threshold Warning

**Trigger**: Cost reaches `CONFIG.WARN_AT_PERCENT` of budget

**Format**:
```
⚠️  BUDGET THRESHOLD WARNING
Current cost: $0.80 / $1.00 (80% of budget used)
Estimated remaining operations: ~$0.25
You may exceed budget soon.

Continue? [Y/n]
```

**Behavior**:
- Shown only once when threshold crossed
- Non-blocking (can continue regardless)
- Helps user be aware of budget status

**Example**:
```cheesecake
CONFIG:
  BUDGET: $1.00
  WARN_AT_PERCENT: 80
END CONFIG

# When accumulated cost reaches $0.80, warning appears
```

### 4. Budget Exceeded Error

**Trigger**: Operation would cause total cost to exceed `CONFIG.BUDGET`

**Format**:
```
❌ BUDGET EXCEEDED
Current cost: $0.95
Next operation estimated: ~$0.15
Total would be: $1.10 (exceeds $1.00 budget)

Operation: RUN SESSION(opus_agent): "Final analysis"
Location: line 45

Workflow stopped to prevent budget overrun.

Options:
1. Increase budget: Modify CONFIG.BUDGET
2. Skip operation: Comment out or remove
3. Use cheaper model: Switch Opus → Sonnet
```

**Behavior** (if `STOP_ON_BUDGET_EXCEED: true`):
- Workflow stops immediately
- No further operations executed
- Error message displayed
- Exit code 1

**Behavior** (if `STOP_ON_BUDGET_EXCEED: false`):
- Warning shown but workflow continues
- Exceeding budget is allowed
- Logged for user awareness

**Example**:
```cheesecake
CONFIG:
  BUDGET: $0.50
  STOP_ON_BUDGET_EXCEED: true
END CONFIG

AGENT OpusAgent:
  MODEL: opus
  PROMPT: "Agent"

VAR agent = NEW OpusAgent()

# First session: ~$0.15 (total: $0.15)
VAR r1 = RUN SESSION(agent): TASK: "Task 1"

# Second session: ~$0.15 (total: $0.30)
VAR r2 = RUN SESSION(agent): TASK: "Task 2"

# Third session: ~$0.15 (total: $0.45)
VAR r3 = RUN SESSION(agent): TASK: "Task 3"

# Fourth session: ~$0.15 (would exceed budget)
# ❌ BUDGET EXCEEDED - stops here
VAR r4 = RUN SESSION(agent): TASK: "Task 4"
```

### Warning Customization

**Disable interactive warnings** (auto-continue):
```cheesecake
CONFIG:
  CONFIRM_COST_ABOVE: $0.10
  INTERACTIVE_WARNINGS: false  # Log but don't block
END CONFIG
```

**Disable all warnings**:
```cheesecake
CONFIG:
  # No CONFIRM_COST_ABOVE
  # No WARN_PARALLEL_ABOVE
  # No budget warnings
END CONFIG
```

---

## Budget Enforcement

### Hard Budget Limit

When `CONFIG.BUDGET` is set and `STOP_ON_BUDGET_EXCEED: true` (default):

**Behavior**:
1. Before each operation, VM checks: `current_cost + estimated_operation_cost <= budget`
2. If check fails, workflow stops immediately
3. Error message displayed with details
4. No partial operations executed
5. Exit with error code

**Example**:
```cheesecake
CONFIG:
  BUDGET: $0.20
  STOP_ON_BUDGET_EXCEED: true
END CONFIG

AGENT Agent:
  MODEL: opus
  PROMPT: "Agent"

VAR agent = NEW Agent()

# Operation 1: ~$0.15 (passes: $0.15 <= $0.20)
VAR r1 = RUN SESSION(agent): TASK: "Task 1"
LOG "Cost so far: $0.15"

# Operation 2: ~$0.15 (fails: $0.15 + $0.15 = $0.30 > $0.20)
# ❌ Workflow stops here
VAR r2 = RUN SESSION(agent): TASK: "Task 2"
```

### Soft Budget Warning

When `STOP_ON_BUDGET_EXCEED: false`:

**Behavior**:
1. Budget can be exceeded
2. Warning shown when exceeded
3. Workflow continues normally
4. Final cost may be over budget
5. Useful for important workflows where completion > cost

**Example**:
```cheesecake
CONFIG:
  BUDGET: $0.20
  STOP_ON_BUDGET_EXCEED: false  # Allow exceeding
END CONFIG

AGENT Agent:
  MODEL: opus
  PROMPT: "Agent"

VAR agent = NEW Agent()

# All operations execute despite exceeding budget
VAR r1 = RUN SESSION(agent): TASK: "Task 1"  # ~$0.15
VAR r2 = RUN SESSION(agent): TASK: "Task 2"  # ~$0.15
VAR r3 = RUN SESSION(agent): TASK: "Task 3"  # ~$0.15

# Final cost: ~$0.45 (exceeds $0.20 budget)
# ⚠️ Warning shown but workflow completes
```

### No Budget Limit

When `CONFIG.BUDGET` is not set:

**Behavior**:
- No budget enforcement
- No budget-related warnings
- Cost still tracked and displayed
- Useful for development/experimentation

**Example**:
```cheesecake
CONFIG:
  CONFIRM_COST_ABOVE: $0.50  # Still warn on expensive ops
  # No BUDGET set
END CONFIG

# No budget limit, but still warns on >$0.50 operations
```

### Budget Management Best Practices

1. **Start conservative**: Set lower budgets initially, increase as needed
2. **Test with dry-run**: Use `--dry-run` to estimate cost before real run
3. **Phase budgets**: Break workflow into phases with checkpoints
4. **Fallback to cheaper models**: Have Sonnet alternatives for Opus tasks
5. **Monitor patterns**: Track which operations consume most budget

---

## Optimization Suggestions

### Overview

The VM analyzes workflow patterns and suggests optimizations to reduce cost while maintaining quality.

**When suggestions appear**:
- After expensive operations complete
- At the end of workflow execution
- When budget threshold warnings triggered
- During dry-run cost estimation

**Control**:
```cheesecake
CONFIG:
  OPTIMIZATION_SUGGESTIONS: true  # Enable (default)
  # or
  OPTIMIZATION_SUGGESTIONS: false  # Disable
END CONFIG
```

### Suggestion Types

#### 1. Model Downgrade Suggestion

**Trigger**: Using expensive model (Opus) for simple/repetitive tasks

**Format**:
```
💡 OPTIMIZATION SUGGESTION #1: Model Downgrade

Location: Line 23
Current: RUN SESSION(opus_agent): "Summarize feedback"
Cost: ~$0.15 per iteration (6 iterations) = ~$0.90

Suggestion: Switch to Sonnet for this task
Potential savings: ~$0.72 (80% reduction)
Quality impact: Minimal (summarization is straightforward)

Recommended change:
  AGENT Summarizer:
-   MODEL: opus
+   MODEL: sonnet
    PROMPT: "..."
```

**When to accept**:
- Task is simple or repetitive
- Quality difference is negligible
- Cost savings are significant

**When to reject**:
- Task requires highest quality (creative writing, complex reasoning)
- Output quality is critical
- Budget is not constrained

#### 2. Parallel Execution Suggestion

**Trigger**: Sequential operations that could run in parallel

**Format**:
```
💡 OPTIMIZATION SUGGESTION #2: Parallelize

Location: Lines 30-35
Current: Sequential execution (3 independent tasks)
Time: ~6-9 seconds
Cost: ~$0.09

Suggestion: Use PARALLEL block
Potential savings:
  - Time: ~3-4 seconds (67% faster)
  - Cost: Same (~$0.09)

Recommended change:
+ PARALLEL:
    VAR r1 = RUN SESSION(agent1): TASK: "Task 1"
    VAR r2 = RUN SESSION(agent2): TASK: "Task 2"
    VAR r3 = RUN SESSION(agent3): TASK: "Task 3"
+ END PARALLEL
```

**Benefits**:
- Faster execution (time savings)
- Same cost (all sessions run anyway)
- Better resource utilization

#### 3. Checkpoint Suggestion

**Trigger**: Long-running workflow without checkpoints

**Format**:
```
💡 OPTIMIZATION SUGGESTION #3: Add Checkpoints

Location: After line 50 (expensive research phase)
Risk: If workflow fails, expensive work is lost
Cost of re-run: ~$0.45

Suggestion: Add checkpoint after research
Benefits:
  - Resume from checkpoint if failure occurs
  - Avoid re-running expensive operations
  - Faster iteration during development

Recommended change:
  VAR research = RUN SESSION(researcher): ...

+ CHECKPOINT "research-complete":
+   SAVE: {research}
+ END CHECKPOINT
```

#### 4. Loop Optimization Suggestion

**Trigger**: Loop with expensive model and uncertain exit

**Format**:
```
💡 OPTIMIZATION SUGGESTION #4: Optimize Loop

Location: Lines 60-65
Current: LOOP UNTIL with Opus model
Max iterations: 10
Estimated cost: ~$1.50 (if all 10 iterations run)

Suggestions:
a) Reduce MAX iterations: 10 → 5
   Savings: ~$0.75 (50% reduction)

b) Switch to Sonnet for iterations
   Savings: ~$1.20 (80% reduction)

c) Use two-phase approach:
   - Sonnet for iterations 1-N
   - Opus for final iteration only
   Savings: ~$1.05 (70% reduction)
```

#### 5. Caching Suggestion

**Trigger**: Repeated identical or similar operations

**Format**:
```
💡 OPTIMIZATION SUGGESTION #5: Cache Results

Location: Lines 40, 52, 68 (3 similar research tasks)
Current: 3 separate research sessions
Total cost: ~$0.30

Observation: All three sessions research "AI trends 2025"
Suggestion: Consolidate into single session, reuse results

Savings: ~$0.20 (67% reduction)

Recommended change:
+ VAR shared_research = RUN SESSION(researcher):
+   TASK: "Comprehensive research on AI trends 2025"
+
  # Reuse shared_research for all three downstream tasks
```

#### 6. CHOICE ON Optimization

**Trigger**: Executing unnecessary branches

**Format**:
```
💡 OPTIMIZATION SUGGESTION #6: Use CHOICE ON

Location: Lines 75-90
Current: Both branches execute, only one result used
Wasted cost: ~$0.15

Suggestion: Use CHOICE ON to execute only needed branch

Current:
  VAR detailed = RUN SESSION(opus_agent): ...
  VAR summary = RUN SESSION(sonnet_agent): ...
  IF need_detail:
    RETURN detailed
  ELSE:
    RETURN summary

Optimized:
  CHOICE ON **level of detail needed**:
    OPTION "detailed":
      VAR result = RUN SESSION(opus_agent): ...
    OPTION "summary":
      VAR result = RUN SESSION(sonnet_agent): ...
  END CHOICE
  RETURN result

Savings: ~$0.15 (50% reduction on average)
```

### Suggestion Display

Suggestions appear in a summary section at the end:

```
============================================
OPTIMIZATION SUGGESTIONS
============================================

You have 3 potential optimizations:

💡 #1: Model Downgrade (Line 23)
   Potential savings: ~$0.72 (80%)

💡 #2: Parallelize (Lines 30-35)
   Potential savings: ~4 seconds (67% faster)

💡 #3: Add Checkpoint (Line 50)
   Reduces risk of re-run cost: ~$0.45

Total potential savings: ~$0.72 + time improvements

View details: /cheesecake explain workflow.cheesecake --suggestions
Apply suggestions: /cheesecake optimize workflow.cheesecake
============================================
```

### Applying Suggestions

**Manual**: Read suggestions and modify code
**Semi-automatic** (future): `/ cheesecake optimize <file>` generates optimized version

---

## Integration with Estimation

Cost management integrates closely with Module 9 (cost estimation):

### Estimation Before Execution

1. **Dry-run**: `/cheesecake run --dry-run workflow.cheesecake`
   - Shows estimated cost breakdown
   - No actual execution (zero cost)
   - Budget checks simulated

2. **Estimate command**: `/cheesecake estimate workflow.cheesecake`
   - Quick cost analysis
   - Shows total and phase-by-phase estimates
   - Suggests optimizations

### Estimation During Execution

**Before each operation**:
- VM estimates cost based on:
  - Model type (Sonnet/Opus/Haiku)
  - Task complexity (simple/complex)
  - Context size
  - Estimated output length
- Uses estimation formulas from Module 9
- Compares against budget

**Cost tracking**:
- Estimated cost shown before operation
- Actual cost recorded after operation
- Running total updated
- Variance tracked (estimate vs actual)

### Estimation Accuracy

**Factors affecting accuracy**:
- Task complexity (can vary)
- Output length (hard to predict)
- Context size (grows with conversation)
- Model performance (may use more/less tokens)

**Confidence levels**:
- **High (90%)**: Simple tasks, small context, predictable output
- **Medium (70%)**: Moderate tasks, medium context
- **Low (50%)**: Complex tasks, large context, variable output

**Buffer**:
- VM adds 20% buffer to estimates for safety
- Budget checks use buffered estimates
- Actual costs usually within or below estimate

### Integration Example

```cheesecake
CONFIG:
  BUDGET: $1.00
  CONFIRM_COST_ABOVE: $0.10
END CONFIG

AGENT Researcher:
  MODEL: opus
  PROMPT: "Researcher"

VAR researcher = NEW Researcher()

# Estimation phase (before execution):
# - VM estimates: ~$0.15 (Opus, complex task)
# - Budget check: $0.00 + $0.15 = $0.15 <= $1.00 ✓
# - Cost warning: $0.15 > $0.10 → Show warning

# User confirms → Execution phase:
VAR result = RUN SESSION(researcher):
  TASK: "Comprehensive market analysis"

# Actual tracking (after execution):
# - Actual cost: $0.14 (slightly under estimate)
# - Running total: $0.14
# - Variance: -$0.01 (7% under estimate)
# - Confidence: High (estimate was accurate)
```

---

## Examples

### Example 1: Basic Budget Control

```cheesecake
# ============================================
# Example 1: Basic Budget Control
# Workflow with simple budget limit
# ============================================

CONFIG:
  BUDGET: $0.50
  CONFIRM_COST_ABOVE: $0.10
END CONFIG

AGENT Researcher:
  MODEL: sonnet
  PROMPT: "You are a researcher."

AGENT Writer:
  MODEL: opus
  PROMPT: "You are a writer."

VAR researcher = NEW Researcher()
VAR writer = NEW Writer()

# Research phase (~$0.03, no warning)
VAR findings = RUN SESSION(researcher):
  TASK: "Research AI trends in 2025"

# Writing phase (~$0.15, triggers warning)
# User must confirm before this executes
VAR article = RUN SESSION(writer):
  TASK: "Write comprehensive 1000-word article on AI trends"
  INPUT: {findings}

# Total cost: ~$0.18 (within $0.50 budget)
SAVE article TO "output/article.md"
```

**Output**:
```
Running: example1.cheesecake

✓ Research phase complete (~$0.03)

⚠️  COST WARNING
This operation estimated at: ~$0.15
Model: opus
Current budget: $0.03 / $0.50 (6%)
After operation: ~$0.18 / $0.50 (36%)
Continue? [Y/n] Y

→ Writing phase...
✓ Writing phase complete (~$0.15)

✓ Workflow complete
Total cost: $0.18 / $0.50 budget (36% used)
```

---

### Example 2: Parallel Session Warning

```cheesecake
# ============================================
# Example 2: Parallel Session Warning
# Process many items with warning on high count
# ============================================

CONFIG:
  BUDGET: $1.00
  WARN_PARALLEL_ABOVE: 5
  OPTIMIZATION_SUGGESTIONS: true
END CONFIG

AGENT Processor:
  MODEL: sonnet
  PROMPT: "You process data items."

VAR processor = NEW Processor()

VAR items = [
  "Item 1", "Item 2", "Item 3", "Item 4", "Item 5",
  "Item 6", "Item 7", "Item 8", "Item 9", "Item 10"
]

PRINT "Processing {items.length} items in parallel..."

# This triggers warning (10 > 5)
PARALLEL FOR item IN items:
  VAR result = RUN SESSION(processor):
    TASK: "Process: {item}"
END PARALLEL FOR

PRINT "✓ All items processed"
```

**Output**:
```
Running: example2.cheesecake

Processing 10 items in parallel...

⚠️  PARALLEL SESSION WARNING
About to spawn 10 parallel sessions
Model: sonnet (x10)
Estimated total cost: ~$0.30
Current budget: $0.00 / $1.00 (0%)
After operation: ~$0.30 / $1.00 (30%)

Continue? [Y/n/r] Y

→ Spawning 10 parallel sessions...
✓ All 10 sessions complete (~$0.30)

✓ All items processed

Total cost: $0.30 / $1.00 budget (30% used)

============================================
OPTIMIZATION SUGGESTIONS
============================================

💡 #1: Consider Batching (Line 25)
   Current: 10 parallel sessions (~$0.30)
   Suggestion: Batch 10 items into 2 sessions (~$0.06)
   Savings: ~$0.24 (80% reduction)
============================================
```

---

### Example 3: Budget Exceeded Scenario

```cheesecake
# ============================================
# Example 3: Budget Exceeded Scenario
# Demonstrates hard budget enforcement
# ============================================

CONFIG:
  BUDGET: $0.20
  STOP_ON_BUDGET_EXCEED: true
END CONFIG

AGENT OpusAgent:
  MODEL: opus
  PROMPT: "High-quality agent"

VAR agent = NEW OpusAgent()

PRINT "Running 3 Opus sessions (each ~$0.15)..."

# Session 1: ~$0.15 (passes)
PRINT "Session 1..."
VAR r1 = RUN SESSION(agent):
  TASK: "Task 1: Analyze market trends"
PRINT "✓ Session 1 complete (~$0.15)"

# Session 2: ~$0.15 (would exceed $0.20 budget)
# ❌ Workflow stops here
PRINT "Session 2..."
VAR r2 = RUN SESSION(agent):
  TASK: "Task 2: Analyze competitor landscape"

# This code never executes
PRINT "Session 3..."
VAR r3 = RUN SESSION(agent):
  TASK: "Task 3: Generate recommendations"
```

**Output**:
```
Running: example3.cheesecake

Running 3 Opus sessions (each ~$0.15)...
Session 1...
✓ Session 1 complete (~$0.15)

Session 2...

❌ BUDGET EXCEEDED
Current cost: $0.15
Next operation estimated: ~$0.15
Total would be: $0.30 (exceeds $0.20 budget)

Operation: RUN SESSION(agent): "Task 2: Analyze competitor landscape"
Location: line 24

Workflow stopped to prevent budget overrun.

Options:
1. Increase budget: CONFIG: BUDGET: $0.50
2. Use cheaper model: Switch Opus → Sonnet (saves ~80%)
3. Reduce operations: Comment out non-essential tasks

Total cost: $0.15 / $0.20 budget (75% used)
```

---

### Example 4: Multi-Phase with Optimization

```cheesecake
# ============================================
# Example 4: Multi-Phase with Optimization
# Complex workflow with phases and suggestions
# ============================================

CONFIG:
  BUDGET: $2.00
  CONFIRM_COST_ABOVE: $0.25
  WARN_AT_PERCENT: 75
  OPTIMIZATION_SUGGESTIONS: true
END CONFIG

AGENT Researcher:
  MODEL: sonnet
  PROMPT: "Researcher"

AGENT Analyst:
  MODEL: opus
  PROMPT: "Deep analyst"

AGENT Writer:
  MODEL: opus
  PROMPT: "Creative writer"

PHASE "Research":
  VAR researcher = NEW Researcher()

  # 3 parallel research tasks
  PARALLEL:
    VAR r1 = RUN SESSION(researcher): TASK: "Research topic A"
    VAR r2 = RUN SESSION(researcher): TASK: "Research topic B"
    VAR r3 = RUN SESSION(researcher): TASK: "Research topic C"
  END PARALLEL

  PRINT "✓ Research complete (~$0.09)"

  CHECKPOINT "research-done":
    SAVE: {r1, r2, r3}
  END CHECKPOINT
END PHASE

PHASE "Analysis":
  VAR analyst = NEW Analyst()

  # Expensive Opus analysis
  VAR analysis = RUN SESSION(analyst):
    TASK: "Deep analysis of research findings"
    INPUT: {r1, r2, r3}

  PRINT "✓ Analysis complete (~$0.30)"

  CHECKPOINT "analysis-done":
    SAVE: {analysis}
  END CHECKPOINT
END PHASE

PHASE "Writing":
  VAR writer = NEW Writer()

  # Iterative writing with Opus
  VAR draft = RUN SESSION(writer):
    TASK: "Write article"
    INPUT: {analysis}

  # Up to 5 refinement iterations
  LOOP UNTIL **{draft} is publication-ready** MAX 5:
    VAR feedback = RUN SESSION(writer):
      TASK: "Review and suggest improvements"
      INPUT: {draft}

    VAR draft = RUN SESSION(writer):
      TASK: "Apply improvements"
      INPUT: {draft, feedback}
  END LOOP

  PRINT "✓ Writing complete"
  SAVE draft TO "output/article.md"
END PHASE
```

**Output**:
```
Running: example4.cheesecake

[■■□□□□] 30% - Phase: Research
✓ Research complete (~$0.09)
Cost: $0.09 / $2.00 (4%)

[■■■■□□] 60% - Phase: Analysis

⚠️  COST WARNING
This operation estimated at: ~$0.30
Model: opus
Current budget: $0.09 / $2.00 (4%)
After operation: ~$0.39 / $2.00 (20%)
Continue? [Y/n] Y

✓ Analysis complete (~$0.30)
Cost: $0.39 / $2.00 (20%)

[■■■■■■] 90% - Phase: Writing
→ Iteration 1...
→ Iteration 2...
→ Iteration 3...
✓ Draft ready
✓ Writing complete
Cost: $1.45 / $2.00 (73%)

✓ Workflow complete
Total cost: $1.45 / $2.00 budget (73% used)

============================================
OPTIMIZATION SUGGESTIONS
============================================

💡 #1: Model Downgrade for Iterations (Line 65)
   Current: Opus for all 5 refinement iterations (~$0.75)
   Suggestion: Use Sonnet for iterations, Opus for final pass
   Savings: ~$0.60 (80% reduction)

   Recommended:
   - Iterations 1-4: Sonnet (~$0.12)
   - Iteration 5: Opus final polish (~$0.15)

💡 #2: Good Use of Checkpoints
   ✓ You're already using checkpoints effectively!
   This prevents re-running expensive phases.

Total potential savings: ~$0.60
============================================
```

---

### Example 5: Soft Budget (Warning Only)

```cheesecake
# ============================================
# Example 5: Soft Budget (Warning Only)
# Allow exceeding budget with warnings
# ============================================

CONFIG:
  BUDGET: $0.30
  STOP_ON_BUDGET_EXCEED: false  # Warn but don't stop
  WARN_AT_PERCENT: 80
END CONFIG

AGENT OpusAgent:
  MODEL: opus
  PROMPT: "Agent"

VAR agent = NEW OpusAgent()

PRINT "Critical workflow - must complete even if over budget"

# Session 1: ~$0.15 (50% of budget)
VAR r1 = RUN SESSION(agent): TASK: "Critical task 1"
PRINT "✓ Task 1 complete"

# Session 2: ~$0.15 (100% of budget)
VAR r2 = RUN SESSION(agent): TASK: "Critical task 2"
PRINT "✓ Task 2 complete"

# Session 3: ~$0.15 (150% of budget - exceeds!)
# Warning shown but execution continues
VAR r3 = RUN SESSION(agent): TASK: "Critical task 3"
PRINT "✓ Task 3 complete"

PRINT "All critical tasks completed"
```

**Output**:
```
Running: example5.cheesecake

Critical workflow - must complete even if over budget

✓ Task 1 complete (~$0.15)
Cost: $0.15 / $0.30 (50%)

⚠️  BUDGET THRESHOLD WARNING
Current cost: $0.24 / $0.30 (80% of budget used)
Next operation estimated: ~$0.15
Total would be: ~$0.39 (exceeds budget)
Continue? [Y/n] Y

✓ Task 2 complete (~$0.15)
Cost: $0.30 / $0.30 (100%)

⚠️  BUDGET EXCEEDED WARNING
Current cost: $0.30
Next operation estimated: ~$0.15
Total would be: $0.45 (exceeds $0.30 budget)
Workflow will continue (STOP_ON_BUDGET_EXCEED: false)

✓ Task 3 complete (~$0.15)
Cost: $0.45 / $0.30 (150% - OVER BUDGET)

All critical tasks completed

⚠️  FINAL WARNING
Workflow exceeded budget!
Budget: $0.30
Actual cost: $0.45
Overage: $0.15 (50% over)
```

---

## Best Practices

### 1. Start with Dry-Run

**Always** test workflows with `--dry-run` first:

```bash
# Estimate cost before running
/cheesecake run --dry-run workflow.cheesecake

# Review estimates
# Adjust CONFIG if needed
# Then run for real
/cheesecake run workflow.cheesecake
```

**Benefits**:
- Zero cost
- See full cost breakdown
- Catch expensive operations
- Adjust budget accordingly

### 2. Set Realistic Budgets

**Conservative approach**:
1. Run dry-run to get estimate
2. Add 50% buffer for variance
3. Set budget to estimate × 1.5

**Example**:
```
Dry-run estimate: $0.60
Buffer (50%): $0.30
Budget setting: $0.90
```

```cheesecake
CONFIG:
  BUDGET: $0.90  # Estimate ($0.60) + 50% buffer
END CONFIG
```

### 3. Use Confirmation Thresholds

Set `CONFIRM_COST_ABOVE` to catch expensive operations:

**Guideline**:
- Development: $0.50 (permissive)
- Staging: $0.10 (moderate)
- Production: $0.05 (conservative)

```cheesecake
CONFIG:
  CONFIRM_COST_ABOVE: $0.10  # Moderate threshold
END CONFIG
```

### 4. Optimize Model Selection

**Use the right model for the job**:

```cheesecake
# ❌ Bad: Opus for everything
AGENT Worker:
  MODEL: opus  # $$$
  PROMPT: "..."

# ✓ Good: Match model to task complexity
AGENT SimpleWorker:
  MODEL: sonnet  # Simple tasks, cost-effective

AGENT ComplexAnalyst:
  MODEL: opus  # Complex analysis, worth the cost

AGENT BatchProcessor:
  MODEL: haiku  # Repetitive tasks, cheapest
```

### 5. Add Checkpoints Strategically

**Checkpoint after expensive phases**:

```cheesecake
# Expensive research phase
PHASE "Research":
  # ... expensive operations ...

  # ✓ Checkpoint after expensive work
  CHECKPOINT "research-complete":
    SAVE: {findings}
  END CHECKPOINT
END PHASE

# If writing fails, research doesn't need to re-run
PHASE "Writing":
  # ... can resume from checkpoint ...
END PHASE
```

### 6. Batch Instead of Loop

**Batch processing is often cheaper**:

```cheesecake
# ❌ Expensive: Loop with 10 iterations
LOOP 10:
  VAR result = RUN SESSION(agent):
    TASK: "Process item {i}"
END LOOP
# Cost: 10 sessions × $0.03 = $0.30

# ✓ Cheaper: Batch all items in one session
VAR all_results = RUN SESSION(agent):
  TASK: "Process all items: {items}"
# Cost: 1 session × $0.05 = $0.05 (83% savings!)
```

### 7. Use Parallel Wisely

**Parallel is faster but same cost**:

```cheesecake
# Sequential: 3 sessions, slow, ~$0.09
VAR r1 = RUN SESSION(agent): TASK: "Task 1"
VAR r2 = RUN SESSION(agent): TASK: "Task 2"
VAR r3 = RUN SESSION(agent): TASK: "Task 3"

# Parallel: 3 sessions, fast, ~$0.09 (same cost!)
PARALLEL:
  VAR r1 = RUN SESSION(agent): TASK: "Task 1"
  VAR r2 = RUN SESSION(agent): TASK: "Task 2"
  VAR r3 = RUN SESSION(agent): TASK: "Task 3"
END PARALLEL
```

**Use parallel for**:
- Independent operations
- Time-sensitive workflows
- When cost is acceptable

**Avoid parallel for**:
- Operations that can be batched
- When hitting session limits
- When budget is very tight

### 8. Review Optimization Suggestions

**Always review** suggestions at the end:

```cheesecake
CONFIG:
  OPTIMIZATION_SUGGESTIONS: true  # Always enable
END CONFIG

# After workflow completes, review suggestions
# Apply promising optimizations
# Re-run with improvements
```

### 9. Test Budget Limits

**Test budget enforcement** in development:

```cheesecake
CONFIG:
  BUDGET: $0.01  # Intentionally low
  STOP_ON_BUDGET_EXCEED: true
END CONFIG

# Workflow will stop quickly
# Verify error handling works
# Then increase to real budget
```

### 10. Monitor Cost Trends

**Track costs over time**:

```bash
# Run and log costs
/cheesecake run workflow.cheesecake | tee costs.log

# Review trends
grep "Total cost:" costs.log

# Identify cost increases
# Investigate causes
# Optimize accordingly
```

---

## Testing Guidelines

### Test Coverage

Test all cost management features:

1. **CONFIG parsing**: All settings parse correctly
2. **Cost tracking**: Costs accumulate accurately
3. **Operation warnings**: Triggers at correct thresholds
4. **Parallel warnings**: Warns on high session counts
5. **Budget warnings**: Warns at percentage thresholds
6. **Budget enforcement**: Stops when budget exceeded
7. **Soft budget**: Continues when STOP_ON_BUDGET_EXCEED: false
8. **Optimization suggestions**: Generates appropriate suggestions
9. **Integration**: Works with estimation, progress, phases

### Test File Structure

```cheesecake
# ============================================
# Test: Cost Management Features
# Purpose: Validate all Module 11 functionality
# ============================================

# TEST 1: CONFIG Block Parsing
CONFIG:
  BUDGET: $1.00
  CONFIRM_COST_ABOVE: $0.10
  WARN_PARALLEL_ABOVE: 5
  DEFAULT_MODEL: sonnet
  OPTIMIZATION_SUGGESTIONS: true
END CONFIG

# Validation: CONFIG parses without errors
PRINT "✓ TEST 1: CONFIG parsed successfully"

# TEST 2: Cost Tracking
# ... test actual cost tracking ...

# TEST 3: Budget Enforcement
# ... test budget limits ...

# Continue for all features...
```

### Testing Strategies

**1. Unit tests**: Test individual features

```cheesecake
# Test: Operation cost warning
CONFIG:
  CONFIRM_COST_ABOVE: $0.10
END CONFIG

AGENT OpusAgent:
  MODEL: opus
  PROMPT: "Agent"

VAR agent = NEW OpusAgent()

# Expected: Warning appears (~$0.15 > $0.10)
VAR result = RUN SESSION(agent):
  TASK: "Expensive task"

# Validation: User was prompted
PRINT "✓ Cost warning triggered correctly"
```

**2. Integration tests**: Test feature interactions

```cheesecake
# Test: Budget enforcement with phases
CONFIG:
  BUDGET: $0.20
END CONFIG

PHASE "Phase 1":
  # ... operations totaling ~$0.15 ...
END PHASE

PHASE "Phase 2":
  # ... operations totaling ~$0.15 ...
  # Expected: Budget exceeded, stops here
END PHASE

# Validation: Phase 2 never executes
```

**3. Regression tests**: Ensure backward compatibility

```cheesecake
# Test: Workflow without CONFIG still works
# No CONFIG block at all

AGENT Agent:
  MODEL: sonnet
  PROMPT: "Agent"

VAR agent = NEW Agent()
VAR result = RUN SESSION(agent): TASK: "Task"

# Validation: Works normally, no errors
PRINT "✓ Backward compatibility maintained"
```

### Validation Checklist

- [ ] CONFIG block parses all settings correctly
- [ ] Invalid CONFIG values show clear errors
- [ ] Cost tracking accumulates accurately
- [ ] Estimated costs have reasonable accuracy (±20%)
- [ ] Operation warnings trigger at correct thresholds
- [ ] Parallel warnings count sessions correctly
- [ ] Budget percentage warnings appear once
- [ ] Budget exceeded stops workflow (hard limit)
- [ ] Soft budget allows exceeding with warnings
- [ ] Optimization suggestions are relevant
- [ ] Suggestions don't recommend bad changes
- [ ] Integration with progress display works
- [ ] Integration with estimation works
- [ ] Works with all constructs (LOOP, PARALLEL, etc.)
- [ ] Backward compatible (no CONFIG = no limits)

---

## Summary

Module 11 (Cost Management) provides comprehensive tools for controlling AI workflow costs:

**Core Features**:
- ✅ CONFIG block for global settings
- ✅ Real-time cost tracking
- ✅ Warnings before expensive operations
- ✅ Hard and soft budget enforcement
- ✅ AI-powered optimization suggestions

**Integration**:
- ✅ Works with Module 9 (estimation)
- ✅ Integrates with progress tracking
- ✅ Compatible with all CheeseCake constructs
- ✅ Backward compatible (optional CONFIG)

**Benefits**:
- 💰 Prevent cost overruns
- 📊 Informed decision making
- ⚡ Optimization opportunities
- 🎯 Budget planning
- 🔍 Cost visibility

**Best Practices**:
1. Always dry-run first
2. Set realistic budgets
3. Use confirmation thresholds
4. Optimize model selection
5. Add strategic checkpoints
6. Review optimization suggestions
7. Test budget limits
8. Monitor cost trends

With cost management, CheeseCake workflows are production-ready with full cost control! 💪

---

**End of Cost Management Specification**
