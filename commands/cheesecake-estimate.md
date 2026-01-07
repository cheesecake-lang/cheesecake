# CheeseCake Estimate Command
# Purpose: Estimate cost and resource usage before workflow execution
# Part of: CheeseCake v0.0.2 - Module 9 (Progress & Dry-Run)
#
# This command analyzes a .cheesecake file and provides cost/resource estimates
# WITHOUT executing it. It's faster than --dry-run and focused on cost analysis.
#
# Usage:
#   /cheesecake estimate <filename>
#   /cheesecake estimate <file1> <file2> (compare costs)
#
# Dependencies:
#   - Read tool (to read .cheesecake file)
#   - cost-estimation.md (for calculation formulas)
#   - SKILL.md (for syntax parsing)
#
# Related:
#   - cheesecake-run.md (execution with --dry-run)
#   - cost-estimation.md (detailed cost formulas)

---
description: Estimate cost and resource usage for a workflow
allowed-tools: Read, Glob
---

# CheeseCake Estimate Command

Quickly estimate the cost, tokens, and time required to run a `.cheesecake` workflow **without executing it**.

## Usage

```
/cheesecake estimate <filename>
```

or

```
/cheesecake estimate <file1> <file2>
```
(to compare multiple workflows)

## What This Command Does

The `estimate` command:
1. **Reads and parses** the .cheesecake file
2. **Analyzes structure** (agents, sessions, loops, parallel blocks)
3. **Calculates estimates** using cost-estimation.md formulas
4. **Shows breakdown** by agent type and phase
5. **Suggests optimizations** if expensive patterns detected

**Key Difference from `--dry-run`:**
- `estimate`: Quick cost analysis only (no step-by-step simulation)
- `--dry-run`: Full execution simulation with progress updates

## Execution Protocol

### Step 1: Locate and Read File

1. If filename provided:
   - Check if file exists
   - If not found, check with `.cheesecake` extension added
   - If still not found, show error and list available files

2. If multiple files provided:
   - Read all files for comparison mode

3. Use Read tool to load file contents

### Step 2: Parse and Analyze

Parse the .cheesecake file to extract:

**Agent Definitions:**
- Count of agents defined
- Model type for each agent (sonnet, opus, haiku)
- Skills used by each agent

**Session Executions:**
- Count of `RUN SESSION()` statements
- Estimate complexity (simple vs complex) based on:
  - Task description length
  - Input data size
  - Context provided
  - Skills required

**Control Flow:**
- PARALLEL blocks (how many concurrent sessions)
- Loops:
  - REPEAT N: Fixed count
  - FOR: Count from list length
  - LOOP UNTIL: Estimate iterations (use MAX/2 as realistic estimate)
- Conditionals (IF/ELIF/ELSE): Average cost of branches

**Phases:**
- Detect PHASE blocks if present
- Group operations by phase for breakdown

### Step 3: Calculate Estimates

Using formulas from `cost-estimation.md`:

#### 3.1 Token Estimation

For each SESSION execution:
```
Simple task:
  Sonnet: 300-800 tokens
  Opus: 500-1200 tokens
  Haiku: 200-500 tokens

Complex task:
  Sonnet: 1500-3500 tokens
  Opus: 2500-6000 tokens
  Haiku: 800-1500 tokens
```

Complexity indicators:
- **Simple**: Short task (<100 chars), no context, basic skills
- **Complex**: Long task, requires research/analysis, multiple skills

#### 3.2 Cost Calculation

```
# Model pricing (2025 rates)
Sonnet 4.5:
  Input: $3.00 per million tokens
  Output: $15.00 per million tokens

Opus 4.5:
  Input: $15.00 per million tokens
  Output: $75.00 per million tokens

Haiku 3.5:
  Input: $0.80 per million tokens
  Output: $4.00 per million tokens

# Per-session cost formula
session_cost = (input_tokens × input_price) + (output_tokens × output_price)

# Total workflow cost
total_cost = Σ(all_session_costs) × 1.2  # 20% buffer
```

#### 3.3 Time Estimation

```
# Average session execution times
Sonnet (simple): 2-3s
Sonnet (complex): 4-8s
Opus (simple): 3-5s
Opus (complex): 6-12s
Haiku (simple): 1-2s
Haiku (complex): 2-4s

# Parallel blocks: Max time of concurrent sessions (not sum)
# Sequential: Sum of all session times
```

### Step 4: Display Estimates

Show comprehensive estimate report:

```
╔═══════════════════════════════════════════════════════════╗
║  Cost Estimate: workflow.cheesecake                       ║
╚═══════════════════════════════════════════════════════════╝

📊 Analysis:
  • 2 agents defined
  • 8 total sessions estimated
  • 1 parallel block (3 concurrent)
  • 1 loop (max 5 iterations, est. 3 actual)

─────────────────────────────────────────────────────────────
💰 COST ESTIMATE:

Sessions by model:
  • 5 Sonnet sessions (research & analysis)      ~$0.08
  • 3 Opus sessions (writing & review)           ~$0.35
                                                 ──────
Total estimated cost: $0.43
Range: $0.30 - $0.65 (depends on loop iterations)

─────────────────────────────────────────────────────────────
📈 RESOURCE BREAKDOWN:

Tokens: ~18,000 total
  • Input: ~12,000 tokens
  • Output: ~6,000 tokens

Time: ~35-50s
  • Sequential phases: ~25s
  • Parallel research: ~8s (3 sessions concurrently)

─────────────────────────────────────────────────────────────
💡 OPTIMIZATION SUGGESTIONS:

1. ⚠️ Writing phase uses Opus (85% of cost)
   → Consider using Sonnet for drafts, Opus for final polish
   → Potential savings: ~$0.25 (58% reduction)

2. ✓ Good use of parallel execution for research
   → Saves ~10s compared to sequential

─────────────────────────────────────────────────────────────
Next steps:
  • Run with dry-run: /cheesecake run workflow.cheesecake --dry-run
  • Execute workflow: /cheesecake run workflow.cheesecake
─────────────────────────────────────────────────────────────
```

### Step 5: Confidence Level

Include confidence level with estimates:

```
Confidence: High (85%)
  • All loops have fixed bounds
  • No semantic conditions with unknown outcomes
  • Task complexity straightforward to estimate

Confidence: Medium (60%)
  • Contains semantic loop (unknown iterations)
  • Assuming 2-5 iterations based on MAX

Confidence: Low (40%)
  • Multiple conditional branches with unknown probability
  • Task complexity varies significantly based on input
  • Actual cost may differ substantially
```

## Comparison Mode

When multiple files provided:

```
/cheesecake estimate workflow-v1.cheesecake workflow-v2.cheesecake
```

Output:

```
╔═══════════════════════════════════════════════════════════╗
║  Cost Comparison                                          ║
╚═══════════════════════════════════════════════════════════╝

┌─────────────────────────────────────────────────────────┐
│  workflow-v1.cheesecake (All Opus approach)            │
├─────────────────────────────────────────────────────────┤
│  Cost: $1.20                                            │
│  Tokens: ~45,000                                        │
│  Time: ~60s                                             │
│  Sessions: 8 Opus                                       │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│  workflow-v2.cheesecake (Mixed Sonnet/Opus) ⭐          │
├─────────────────────────────────────────────────────────┤
│  Cost: $0.43                                            │
│  Tokens: ~18,000                                        │
│  Time: ~45s                                             │
│  Sessions: 5 Sonnet, 3 Opus                            │
└─────────────────────────────────────────────────────────┘

─────────────────────────────────────────────────────────────
💰 SAVINGS: $0.77 (64% reduction) ✅
⏱️  TIME SAVINGS: ~15s (25% faster)

🏆 WINNER: workflow-v2.cheesecake

Recommendation: workflow-v2 is more cost-efficient without
sacrificing quality by using Sonnet for routine tasks and
Opus for critical final steps.
─────────────────────────────────────────────────────────────
```

## Cost Breakdown by Phase

If workflow uses PHASE blocks, show phase-by-phase breakdown:

```
╔═══════════════════════════════════════════════════════════╗
║  Cost Estimate: research-workflow.cheesecake              ║
╚═══════════════════════════════════════════════════════════╝

📊 Phase-by-Phase Breakdown:

Phase 1: Research (parallel)
  • 3 Sonnet sessions (concurrent)
  • Tokens: ~7,500
  • Cost: ~$0.06
  • Time: ~8s

Phase 2: Analysis
  • 1 Sonnet session
  • Tokens: ~2,500
  • Cost: ~$0.02
  • Time: ~3s

Phase 3: Writing (iterative)
  • 1 Opus initial draft
  • Loop: 2-5 review iterations
  • Tokens: ~15,000-40,000
  • Cost: ~$0.35-0.85
  • Time: ~20-45s

Phase 4: Output
  • File I/O only
  • Cost: $0.00
  • Time: ~0.1s

─────────────────────────────────────────────────────────────
TOTAL ESTIMATE:

Cost: $0.43 (range: $0.30 - $0.93)
Tokens: ~25,000-55,000
Time: ~31-56s

Cost by phase:
  Research:   $0.06  (14%)
  Analysis:   $0.02  (5%)
  Writing:    $0.35  (81%)  ← Most expensive
  Output:     $0.00  (0%)

💡 The writing phase dominates cost due to Opus usage and
iterative refinement. Consider reducing MAX iterations or
using Sonnet for early drafts.
─────────────────────────────────────────────────────────────
```

## Error Handling

### File Not Found

```
❌ Error: File not found: workflow.cheesecake

Available .cheesecake files in current directory:
  • hello.cheesecake
  • research-pipeline.cheesecake
  • customer-feedback-analysis.cheesecake

Usage: /cheesecake estimate <filename>
```

### Invalid Syntax

```
❌ Syntax Error: Cannot estimate due to syntax issues

File: workflow.cheesecake
Line 25: Missing END LOOP statement

Please fix syntax errors first:
  /cheesecake validate workflow.cheesecake

Or use dry-run to see detailed error:
  /cheesecake run workflow.cheesecake --dry-run
```

### Multiple Files Not Found

```
❌ Error: One or more files not found

Found:
  ✓ workflow-v1.cheesecake
  ✗ workflow-v2.cheesecake (not found)

Please check filenames and try again.
```

## Estimation Algorithm Details

### Detecting Task Complexity

The estimate command uses heuristics to determine if a task is "simple" or "complex":

**Simple Task Indicators:**
- Task description < 100 characters
- No INPUT context provided
- No external skills (web-research, data-analysis)
- Simple prompts like "summarize", "format", "extract"

**Complex Task Indicators:**
- Task description > 200 characters
- Multiple inputs or large context
- Skills requiring external resources
- Tasks involving research, analysis, generation

**Example:**
```cheesecake
# Simple task (300-800 tokens estimate)
VAR summary = RUN SESSION(agent):
  TASK: "Summarize this text"

# Complex task (1500-3500 tokens estimate)
VAR analysis = RUN SESSION(researcher):
  TASK: "Research recent quantum computing breakthroughs,
         analyze market impact, and provide strategic recommendations"
  CONTEXT: {domain: "physics", depth: "comprehensive"}
```

### Loop Iteration Estimation

**REPEAT N:**
```cheesecake
REPEAT 5:
  VAR x = RUN SESSION(agent): TASK: "..."
END REPEAT

Estimate: Exactly 5 sessions (high confidence)
```

**FOR loop:**
```cheesecake
VAR items = ["a", "b", "c"]
FOR item IN items:
  VAR result = RUN SESSION(agent): TASK: "Process {item}"
END FOR

Estimate: Exactly 3 sessions (high confidence)
```

**LOOP UNTIL with MAX:**
```cheesecake
LOOP UNTIL **{draft} is perfect** MAX 5:
  VAR draft = RUN SESSION(agent): TASK: "Improve {draft}"
END LOOP

Estimate:
  - Optimistic: 2 iterations
  - Realistic: MAX/2 = 2.5 ≈ 3 iterations (use this)
  - Pessimistic: MAX = 5 iterations
Range: 2-5 sessions (medium confidence)
```

### Parallel Block Costing

```cheesecake
PARALLEL:
  VAR a = RUN SESSION(agent1): TASK: "Task A"  # 3s, $0.02
  VAR b = RUN SESSION(agent2): TASK: "Task B"  # 5s, $0.03
  VAR c = RUN SESSION(agent3): TASK: "Task C"  # 4s, $0.02
END PARALLEL

Cost estimate: $0.02 + $0.03 + $0.02 = $0.07 (all sessions)
Time estimate: MAX(3s, 5s, 4s) = 5s (longest session)
```

### Conditional Branch Estimation

```cheesecake
IF **{analysis} shows growth**:
  VAR report = RUN SESSION(writer): TASK: "Optimistic"  # $0.05
ELSE:
  VAR report = RUN SESSION(writer): TASK: "Cautious"    # $0.04
END IF

Estimate: ($0.05 + $0.04) / 2 = $0.045 (average)
Note: Actual cost depends on which branch executes
```

## Best Practices for Cost Estimation

### 1. Use Estimates Early

Run `estimate` before executing expensive workflows:

```bash
# Check cost first
/cheesecake estimate expensive-workflow.cheesecake

# If acceptable, run it
/cheesecake run expensive-workflow.cheesecake
```

### 2. Compare Approaches

When designing workflows, compare different implementations:

```bash
# Compare all-Opus vs mixed approach
/cheesecake estimate all-opus.cheesecake mixed-models.cheesecake
```

### 3. Iterate on Design

Use estimates to guide optimization:

```bash
# Initial design
/cheesecake estimate workflow-v1.cheesecake
# → Cost: $1.50

# Optimize (reduce Opus usage)
/cheesecake estimate workflow-v2.cheesecake
# → Cost: $0.45 (70% savings)
```

### 4. Understand Confidence Levels

Pay attention to confidence indicators:
- **High confidence**: Estimates are reliable
- **Medium confidence**: Range may be wider than shown
- **Low confidence**: Actual cost may differ significantly

## Integration with Other Commands

### With `/cheesecake run --dry-run`

```bash
# Quick estimate (this command)
/cheesecake estimate workflow.cheesecake
# → Shows: Cost: $0.43, Time: ~45s

# Detailed simulation
/cheesecake run workflow.cheesecake --dry-run
# → Shows: Step-by-step what would happen + costs
```

**When to use each:**
- `estimate`: Quick cost check before running
- `--dry-run`: Detailed execution preview, catch errors

### With `/cheesecake validate`

```bash
# Validate syntax first
/cheesecake validate workflow.cheesecake
# → Checks for syntax errors

# Then estimate cost
/cheesecake estimate workflow.cheesecake
# → Shows cost breakdown
```

### With `/cheesecake run`

```bash
# Typical workflow:
/cheesecake estimate workflow.cheesecake  # Check cost
# → Cost: $0.43 (acceptable)

/cheesecake run workflow.cheesecake       # Execute
```

## Output Format Reference

### Minimal Output (No Phases)

```
╔═══════════════════════════════════════════════════════════╗
║  Cost Estimate: simple-workflow.cheesecake                ║
╚═══════════════════════════════════════════════════════════╝

Cost: $0.08
Tokens: ~3,500
Time: ~12s
Sessions: 4 Sonnet

Confidence: High (90%)
─────────────────────────────────────────────────────────────
```

### Detailed Output (With Phases)

```
╔═══════════════════════════════════════════════════════════╗
║  Cost Estimate: complex-workflow.cheesecake               ║
╚═══════════════════════════════════════════════════════════╝

📊 Analysis:
  • 3 agents defined (Researcher, Analyst, Writer)
  • 12 total sessions estimated
  • 2 parallel blocks
  • 1 loop (max 5, est. 3)

─────────────────────────────────────────────────────────────
💰 COST ESTIMATE:

Phase 1: Research        $0.06  (12%)
Phase 2: Analysis        $0.04  (8%)
Phase 3: Writing         $0.35  (70%)
Phase 4: Review          $0.05  (10%)
                        ──────
Total:                   $0.50

Range: $0.35 - $0.85

─────────────────────────────────────────────────────────────
📈 RESOURCES:

Tokens: ~22,000 (15,000 input, 7,000 output)
Time: ~45-65s
  • Parallel sections save ~15s

Sessions by model:
  • 8 Sonnet (research, analysis)
  • 4 Opus (writing, review)

─────────────────────────────────────────────────────────────
💡 OPTIMIZATIONS:

1. Writing phase is 70% of cost
   → Consider Sonnet for drafts: Save ~$0.20

2. Good parallel structure
   → Already optimized for time

Confidence: Medium (65%)
─────────────────────────────────────────────────────────────
```

## Implementation Notes

### For the Executing Agent

When implementing this command:

1. **Parse the file** using SKILL.md syntax rules
2. **Identify all SESSION executions** and their agents
3. **Calculate estimates** using cost-estimation.md formulas
4. **Format output** according to templates above
5. **Provide actionable insights** (optimization suggestions)

### Cost Calculation Reference

```javascript
// Pseudo-code for cost calculation

function estimate_workflow(cheesecake_file):
  // Parse file
  agents = extract_agents(cheesecake_file)
  sessions = extract_sessions(cheesecake_file)

  total_cost = 0
  total_tokens = 0
  total_time = 0

  for session in sessions:
    // Determine complexity
    complexity = analyze_task_complexity(session.task)

    // Get model from agent
    model = agents[session.agent].model

    // Estimate tokens
    tokens = get_token_estimate(model, complexity)

    // Calculate cost
    cost = calculate_session_cost(model, tokens)

    // Add to totals
    total_cost += cost
    total_tokens += tokens
    total_time += estimate_time(model, complexity)

  // Apply 20% buffer
  total_cost *= 1.2

  return {
    cost: total_cost,
    tokens: total_tokens,
    time: total_time
  }
```

### Token Estimation Table

```
Model      | Simple Task | Complex Task
-----------|-------------|-------------
Sonnet     | 300-800     | 1500-3500
Opus       | 500-1200    | 2500-6000
Haiku      | 200-500     | 800-1500
```

### Time Estimation Table

```
Model      | Simple Task | Complex Task
-----------|-------------|-------------
Sonnet     | 2-3s        | 4-8s
Opus       | 3-5s        | 6-12s
Haiku      | 1-2s        | 2-4s
```

## Examples

### Example 1: Simple Workflow

User: `/cheesecake estimate hello.cheesecake`

Output:
```
╔═══════════════════════════════════════════════════════════╗
║  Cost Estimate: hello.cheesecake                          ║
╚═══════════════════════════════════════════════════════════╝

Cost: $0.01
Tokens: ~800
Time: ~2s
Sessions: 1 Sonnet (simple greeting task)

Confidence: High (95%)

This is a simple workflow - very inexpensive to run.
─────────────────────────────────────────────────────────────
```

### Example 2: Research Workflow

User: `/cheesecake estimate research-workflow.cheesecake`

Output:
```
╔═══════════════════════════════════════════════════════════╗
║  Cost Estimate: research-workflow.cheesecake              ║
╚═══════════════════════════════════════════════════════════╝

📊 Analysis:
  • 2 agents (Researcher, Writer)
  • 5 sessions (3 parallel + 2 sequential)

─────────────────────────────────────────────────────────────
💰 COST ESTIMATE:

Research phase (parallel):
  • 3 Sonnet sessions                              $0.06
Writing phase:
  • 1 Opus draft                                   $0.08
  • 1 Sonnet review                                $0.02
                                                   ──────
Total:                                             $0.16

─────────────────────────────────────────────────────────────
📈 RESOURCES:

Tokens: ~8,500
Time: ~18s (parallel research saves ~8s)

─────────────────────────────────────────────────────────────
💡 OPTIMIZATIONS:

✓ Efficient use of parallel execution
✓ Good model selection (Sonnet for research, Opus for final)

No major optimizations needed.

Confidence: High (85%)
─────────────────────────────────────────────────────────────
```

### Example 3: Iterative Workflow

User: `/cheesecake estimate iterative-refinement.cheesecake`

Output:
```
╔═══════════════════════════════════════════════════════════╗
║  Cost Estimate: iterative-refinement.cheesecake           ║
╚═══════════════════════════════════════════════════════════╝

📊 Analysis:
  • 1 agent (Writer - Opus)
  • 1 loop (max 10 iterations, unknown exit condition)

─────────────────────────────────────────────────────────────
💰 COST ESTIMATE:

Initial draft:                                     $0.08
Refinement loop (est. 5 iterations):               $0.40
                                                   ──────
Total:                                             $0.48
Range: $0.15 - $0.85 (depends on loop iterations)

⚠️  This workflow has high cost variability due to semantic
    loop condition. Actual cost depends on how many iterations
    are needed to satisfy **{draft} is perfect**.

─────────────────────────────────────────────────────────────
📈 RESOURCES:

Tokens: ~20,000-45,000 (depends on iterations)
Time: ~30-90s

─────────────────────────────────────────────────────────────
💡 OPTIMIZATIONS:

1. ⚠️ Loop uses Opus for all iterations (expensive)
   → Consider: Sonnet for early iterations, Opus for final
   → Potential savings: ~$0.25 (52%)

2. ⚠️ Loop may run up to 10 times
   → Consider: Lower MAX to 5 or add explicit quality check
   → Risk mitigation for runaway costs

Confidence: Low (45%)
  • Semantic exit condition makes iteration count unpredictable
─────────────────────────────────────────────────────────────
```

## Summary

The `/cheesecake estimate` command provides:

✅ **Quick cost analysis** without execution
✅ **Token and time estimates** for planning
✅ **Phase-by-phase breakdown** for complex workflows
✅ **Optimization suggestions** to reduce costs
✅ **Comparison mode** to evaluate different approaches
✅ **Confidence levels** to understand estimate reliability

Use this command **before running expensive workflows** to make informed decisions about execution!
