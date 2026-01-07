# CheeseCake Cost Estimation Specification
# Purpose: Define how to calculate and estimate costs before workflow execution
# Part of: CheeseCake v0.0.2 - Module 9 (Progress & Dry-Run)
#
# This file specifies how to estimate the cost of running a CheeseCake workflow
# before actually executing it, helping users make informed decisions.
#
# Usage:
#   Referenced by cheesecake-estimate.md command
#   Used in dry-run mode to show cost projections
#   Used for budget warnings in cost-management.md
#
# Dependencies:
#   - SKILL.md (language specification for parsing)
#   - vm.md (execution semantics)
#
# Related:
#   - progress.md (token tracking)
#   - cost-management.md (budget enforcement)

---

## Overview

Cost estimation allows users to:
- **Preview costs** before running expensive workflows
- **Compare approaches** to find the most cost-effective
- **Set budgets** and get warnings before exceeding them
- **Make informed decisions** about workflow design

### Core Principles

1. **Conservative estimates** - Better to overestimate than surprise users
2. **Transparent calculation** - Show how costs were derived
3. **Range-based** - Provide min/max estimates when uncertain
4. **Model-aware** - Different models have different costs

---

## Cost Components

### 1. Model Pricing (2025 Rates)

**Claude Sonnet 4.5:**
- Input: $3.00 per million tokens
- Output: $15.00 per million tokens
- Typical session: ~$0.01-0.05

**Claude Opus 4.5:**
- Input: $15.00 per million tokens
- Output: $75.00 per million tokens
- Typical session: ~$0.05-0.25

**Claude Haiku 3.5:**
- Input: $0.80 per million tokens
- Output: $4.00 per million tokens
- Typical session: ~$0.002-0.01

### 2. Token Estimation by Operation

| Operation | Input Tokens | Output Tokens | Total Est. |
|-----------|--------------|---------------|------------|
| Simple task (Sonnet) | 200-500 | 100-300 | 300-800 |
| Complex task (Sonnet) | 1000-2000 | 500-1500 | 1500-3500 |
| Simple task (Opus) | 300-700 | 200-500 | 500-1200 |
| Complex task (Opus) | 1500-3000 | 1000-3000 | 2500-6000 |
| Agent definition | 50-100 | 0 | 50-100 |
| Variable assignment | 10-20 | 0 | 10-20 |
| PRINT statement | 20-50 | 0 | 20-50 |

---

## Estimation Algorithm

### Step 1: Parse the Workflow

Analyze the `.cheesecake` file to identify:
- Agent definitions and their models
- Number of SESSION executions
- PARALLEL blocks (concurrent sessions)
- Loops (iterations × inner cost)
- Conditional branches (probabilistic cost)

### Step 2: Calculate Base Costs

For each operation, calculate cost:

```
operation_cost = (input_tokens × input_price) + (output_tokens × output_price)
```

**Example (Sonnet session):**
```
input_tokens: 500
output_tokens: 300

input_cost = 500 × ($3.00 / 1,000,000) = $0.0015
output_cost = 300 × ($15.00 / 1,000,000) = $0.0045
total_cost = $0.0015 + $0.0045 = $0.006
```

### Step 3: Account for Multipliers

**Loops:**
```
loop_cost = iterations × inner_operation_cost
```

**PARALLEL blocks:**
```
parallel_cost = Σ(all_concurrent_sessions)
```

**Conditional branches:**
```
estimated_cost = probability_branch_a × cost_a + probability_branch_b × cost_b
```

If unknown, use: `(cost_a + cost_b) / 2` (average)

### Step 4: Add Overhead

```
total_cost = base_cost × 1.2  # 20% buffer for context, retries, etc.
```

---

## Detailed Estimation Examples

### Example 1: Simple Hello World

```cheesecake
AGENT Greeter:
  MODEL: sonnet
  PROMPT: "You are a friendly greeter."

VAR greeter = NEW Greeter()

VAR message = RUN SESSION(greeter):
  TASK: "Say hello!"

PRINT message
```

**Estimation:**
```
Operations:
  1. AGENT definition          :    50 tokens  (overhead)
  2. VAR greeter = NEW         :    20 tokens  (overhead)
  3. RUN SESSION (simple)      :   800 tokens  (Sonnet, simple task)
  4. PRINT message             :    30 tokens  (overhead)

Total tokens: 900
Model: Sonnet ($3 in / $15 out per million)

Estimated cost:
  Input:  ~600 tokens × $0.000003 = $0.0018
  Output: ~300 tokens × $0.000015 = $0.0045
  Total: ~$0.0063

With 20% buffer: $0.0076 ≈ $0.01
```

**Display:**
```
Estimated cost: $0.01
  • 1 Sonnet session (simple task)
  • ~900 tokens total
```

### Example 2: Parallel Research

```cheesecake
AGENT Researcher:
  MODEL: sonnet
  SKILLS: [web-research]

VAR researcher = NEW Researcher()

PARALLEL:
  VAR research1 = RUN SESSION(researcher): TASK: "Research topic A"
  VAR research2 = RUN SESSION(researcher): TASK: "Research topic B"
  VAR research3 = RUN SESSION(researcher): TASK: "Research topic C"
END PARALLEL
```

**Estimation:**
```
Operations:
  1. AGENT definition          :    80 tokens  (with skills)
  2. VAR researcher = NEW      :    20 tokens
  3. PARALLEL block:
     - Session 1 (complex)     :  2500 tokens
     - Session 2 (complex)     :  2500 tokens
     - Session 3 (complex)     :  2500 tokens
     Total PARALLEL            :  7500 tokens

Total tokens: 7,600
Model: Sonnet

Estimated cost:
  Input:  ~5,000 tokens × $0.000003 = $0.015
  Output: ~2,600 tokens × $0.000015 = $0.039
  Total: ~$0.054

With buffer: $0.065 ≈ $0.07
```

**Display:**
```
Estimated cost: $0.05-0.07
  • 3 Sonnet sessions in parallel (complex tasks)
  • ~7,600 tokens total
  • Note: All 3 sessions run concurrently
```

### Example 3: Iterative Loop

```cheesecake
AGENT Writer:
  MODEL: opus
  PROMPT: "You are a writer."

VAR writer = NEW Writer()
VAR draft = RUN SESSION(writer): TASK: "Write draft"

LOOP UNTIL **{draft} is perfect** MAX 5:
  VAR feedback = RUN SESSION(writer): TASK: "Review {draft}"
  VAR draft = RUN SESSION(writer): TASK: "Improve based on {feedback}"
END LOOP
```

**Estimation:**
```
Operations:
  1. AGENT definition          :    50 tokens
  2. VAR writer = NEW          :    20 tokens
  3. Initial draft session     :  5000 tokens (Opus, complex)
  4. Loop (estimate 3 iterations):
     - Feedback session        :  3000 tokens × 3 = 9,000
     - Improvement session     :  4000 tokens × 3 = 12,000
     Loop total                : 21,000 tokens

Total tokens: 26,070
Model: Opus

Estimated cost:
  Input:  ~17,000 tokens × $0.000015 = $0.255
  Output:  ~9,000 tokens × $0.000075 = $0.675
  Total: ~$0.930

With buffer: $1.12

Range (1-5 iterations): $0.35 - $2.50
```

**Display:**
```
Estimated cost: $0.35 - $2.50 (depends on loop iterations)
  • 1 Opus session (initial draft)
  • Loop: 2-10 Opus sessions (review + improvement)
  • Max iterations: 5
  • Tokens: ~5,000 - 40,000

💡 Tip: Using Sonnet instead of Opus could save ~80%
```

---

## Cost Estimation by Construct

### AGENT Definitions

```
cost = 0  # No runtime cost, just definition
tokens = 50-100 (overhead for storing definition)
```

### Variable Assignments

```
VAR x = value
cost = 0
tokens = 10-20 (overhead)
```

### SESSION Execution

```
RUN SESSION(agent): TASK: "..."

Sonnet (simple):    $0.005 - $0.015
Sonnet (complex):   $0.020 - $0.050
Opus (simple):      $0.030 - $0.080
Opus (complex):     $0.100 - $0.300
```

### SEQUENCE Block

```
SEQUENCE:
  operation1
  operation2
  operation3
END SEQUENCE

cost = sum(operation1, operation2, operation3)
```

### PARALLEL Block

```
PARALLEL:
  operation1
  operation2
  operation3
END PARALLEL

cost = sum(operation1, operation2, operation3)
# Note: Happens concurrently but costs are additive
```

### FOR Loop

```
FOR item IN list:
  operation
END FOR

cost = len(list) × operation_cost
```

### LOOP UNTIL

```
LOOP UNTIL **condition** MAX N:
  operation
END LOOP

Optimistic estimate: 2 × operation_cost
Realistic estimate: (MAX/2) × operation_cost
Pessimistic estimate: MAX × operation_cost
```

### IF/ELIF/ELSE

```
IF **condition**:
  operation_a
ELSE:
  operation_b
END IF

Estimate (unknown probability):
  cost = (operation_a_cost + operation_b_cost) / 2
```

### CHECKPOINT

```
CHECKPOINT "name":
  SAVE: {data}
END CHECKPOINT

cost = 0  # No AI cost, just file I/O
tokens = 50-100 (overhead)
```

---

## Dry-Run Mode

### What Dry-Run Does

Simulates workflow execution WITHOUT spawning actual AI sessions:

```
/cheesecake run workflow.cheesecake --dry-run
```

**Process:**
1. Parse the .cheesecake file
2. Validate syntax
3. Walk through execution logic
4. Calculate costs at each step
5. Display what WOULD happen
6. Show total estimated cost

**Does NOT:**
- Actually spawn agents
- Execute SESSION tasks
- Make API calls
- Incur any costs

### Dry-Run Output Example

```
╔═══════════════════════════════════════════════════════════╗
║  DRY RUN: research-workflow.cheesecake                    ║
╚═══════════════════════════════════════════════════════════╝

Simulating execution (no actual sessions spawned)...

[■■■■■■■■■■] 100% simulated

✓ Phase 1: Research
  • Would create Researcher agent (Sonnet)
  • Would spawn 3 parallel sessions
  • Estimated: 7,500 tokens, ~$0.06

✓ Phase 2: Analysis
  • Would create Analyst agent (Sonnet)
  • Would run 1 analysis session
  • Estimated: 2,500 tokens, ~$0.02

✓ Phase 3: Writing
  • Would create Writer agent (Opus)
  • Would run 1 writing session
  • Would enter loop (est. 3 iterations)
  • Estimated: 15,000 tokens, ~$0.45

✓ Phase 4: Output
  • Would save to output/article.md
  • No cost (file I/O only)

─────────────────────────────────────────────────────────────
COST ESTIMATE:

Sessions:
  • 3 Sonnet (parallel research):     $0.06
  • 1 Sonnet (analysis):              $0.02
  • 4 Opus (writing + iterations):    $0.45

Total estimated: $0.53
Range: $0.35 - $0.85 (depends on loop iterations)

Tokens: ~25,000
Estimated time: ~45s

─────────────────────────────────────────────────────────────
💰 Cost Breakdown:
  Research:   $0.06  (11%)
  Analysis:   $0.02  (4%)
  Writing:    $0.45  (85%)  ← Highest cost

💡 Optimization suggestions:
  • Writing phase uses Opus (expensive)
  • Consider using Sonnet for drafts, Opus for final polish
  • Could save ~$0.30 (67% reduction)

─────────────────────────────────────────────────────────────
Ready to proceed with actual run? [Y/n]
```

---

## Cost Estimation Command

### `/cheesecake estimate`

Dedicated command for cost estimation without dry-run simulation:

```bash
/cheesecake estimate workflow.cheesecake
```

**Output:**
```
╔═══════════════════════════════════════════════════════════╗
║  Cost Estimate: workflow.cheesecake                       ║
╚═══════════════════════════════════════════════════════════╝

Analysis:
  • 2 agents defined (Sonnet, Opus)
  • 5 total sessions estimated
  • 1 loop (max 5 iterations, est. 3 actual)
  • 1 parallel block (3 concurrent sessions)

Estimated Cost: $0.53
  Range: $0.35 - $0.85

Breakdown:
  Sonnet sessions (4):  $0.08  (15%)
  Opus sessions (1):    $0.45  (85%)

Token estimate: ~25,000
Execution time: ~45s

─────────────────────────────────────────────────────────────
Run /cheesecake run workflow.cheesecake --dry-run for detailed simulation
```

---

## Cost Comparison Feature

Compare multiple approaches:

```bash
/cheesecake estimate workflow-v1.cheesecake workflow-v2.cheesecake
```

**Output:**
```
╔═══════════════════════════════════════════════════════════╗
║  Cost Comparison                                          ║
╚═══════════════════════════════════════════════════════════╝

workflow-v1.cheesecake (All Opus):
  • Cost: $1.20
  • Tokens: ~45,000
  • Time: ~60s

workflow-v2.cheesecake (Mixed Sonnet/Opus):
  • Cost: $0.53
  • Tokens: ~25,000
  • Time: ~45s

Savings: $0.67 (56% reduction) ✅
Winner: workflow-v2.cheesecake
```

---

## Accuracy Guidelines

### Factors Affecting Accuracy

**Predictable (high accuracy):**
- Fixed loops (REPEAT N)
- Simple tasks with known complexity
- Workflows without branches

**Variable (medium accuracy):**
- LOOP UNTIL with semantic conditions
- Conditional branches (IF/ELSE)
- Task complexity estimation

**Uncertain (low accuracy):**
- Dynamic loops with complex exit conditions
- Highly context-dependent tasks
- Workflows with many user interactions

### Confidence Levels

Display confidence with estimates:

```
Estimated cost: $0.53
Confidence: High (90%)
  • All operations are deterministic
  • No semantic exit conditions
  • Fixed iteration counts

Estimated cost: $0.35 - $0.85
Confidence: Medium (60%)
  • Contains 1 semantic loop (unknown iterations)
  • Assuming 2-5 iterations

Estimated cost: $0.50 - $2.00
Confidence: Low (40%)
  • Multiple conditional branches
  • Several semantic loops
  • Actual cost highly variable
```

---

## Optimization Suggestions

When estimating, suggest optimizations:

### 1. Model Selection

```
💡 Optimization: Use Sonnet for routine tasks
   Current: Using Opus for all 5 sessions ($1.20)
   Suggested: Use Sonnet for 4 sessions, Opus for 1 ($0.45)
   Savings: $0.75 (63%)
```

### 2. Parallel Efficiency

```
💡 Optimization: Combine parallel sessions
   Current: 10 small parallel sessions ($0.40)
   Suggested: Batch into 3 larger sessions ($0.25)
   Savings: $0.15 (38%)
```

### 3. Loop Optimization

```
💡 Optimization: Add early exit conditions
   Current: Loop may run up to 10 times ($1.50)
   Suggested: Add quality threshold to exit earlier
   Potential savings: $0.60-1.00
```

### 4. Checkpoint Placement

```
💡 Optimization: Add checkpoint before expensive phase
   If workflow fails in writing phase, you'll lose $0.08 of research
   Add checkpoint after research to avoid re-running
```

---

## Budget Warnings

When workflow exceeds budget thresholds:

```
⚠️ BUDGET WARNING

Estimated cost: $2.45
Your budget: $1.00

This workflow will exceed your budget by $1.45 (145%)

Options:
  [1] Proceed anyway
  [2] Cancel
  [3] Optimize workflow (suggestions available)
  [4] Increase budget limit

Choose an option:
```

---

## API for Programmatic Access

Workflows can query costs:

```cheesecake
VAR estimated_cost = ESTIMATE WORKFLOW "expensive-task.cheesecake"

IF **{estimated_cost} exceeds budget**:
  LOG WARNING: "Cost too high, using cheaper alternative"
  VAR result = RUN SESSION(cheap_agent): TASK: "..."
ELSE:
  VAR result = RUN SESSION(expensive_agent): TASK: "..."
END IF
```

---

## Testing Cost Estimation

### Validation Process

1. **Run workflow with actual costs**
2. **Compare to estimate**
3. **Calculate accuracy:**
   ```
   accuracy = 1 - |actual - estimated| / actual
   ```

4. **Tune estimation formulas** based on results

### Target Accuracy

- **Simple workflows:** ±10%
- **Medium workflows:** ±20%
- **Complex workflows:** ±30%

---

## Summary

Cost estimation empowers users to:

✅ **Make informed decisions** before running workflows
✅ **Optimize for cost** by comparing approaches
✅ **Set and enforce budgets** to prevent surprises
✅ **Understand cost drivers** in their workflows
✅ **Test without spending** using dry-run mode

Combined with progress tracking, this makes CheeseCake production-ready! 💰
