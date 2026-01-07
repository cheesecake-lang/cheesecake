# CheeseCake Progress Tracking Specification
# Purpose: Define how to display real-time progress during workflow execution
# Part of: CheeseCake v0.0.2 - Module 9 (Progress & Dry-Run)
#
# This file specifies how the CheeseCake VM should track and display
# execution progress to provide visibility into running workflows.
#
# Usage:
#   Referenced by vm.md for progress reporting behavior
#   Used during execution to show user what's happening
#
# Dependencies:
#   - vm.md (execution semantics)
#   - cost-estimation.md (for token tracking)
#
# Related:
#   - cheesecake-run.md (displays progress during execution)

---

## Overview

Progress tracking makes CheeseCake workflows transparent by showing:
- **What's currently executing** - Which statement/phase is running
- **How much is complete** - Percentage and visual progress bar
- **Resource usage** - Tokens consumed and estimated remaining
- **Time elapsed** - How long the workflow has been running

### Core Principles

1. **Non-intrusive** - Progress display doesn't require code changes
2. **Informative** - Users understand what's happening at a glance
3. **Accurate** - Progress reflects actual completion state
4. **Real-time** - Updates as execution proceeds

---

## Progress Tracking Levels

### Level 1: Statement-by-Statement (Default)

Track progress for each top-level statement:

```
Executing: customer-feedback-analysis.cheesecake

[■■■□□□□□□□] 30% complete (3/10 statements)

Current: Processing feedback loop (line 44)
Elapsed: 15s | Tokens: 1,250

Status: Running...
```

**When to use:** All workflows automatically get this level

### Level 2: Phase-Based (Optional)

When user marks execution phases with `PHASE` blocks:

```
Executing: research-workflow.cheesecake

[■■■■■□□□□□] 50% complete

✓ Phase 1: Data Collection    [DONE]    5.2s
→ Phase 2: Analysis           [RUNNING] 3.1s
○ Phase 3: Report Generation  [PENDING]
○ Phase 4: Output             [PENDING]

Tokens: 3,420 used | ~3,500 remaining
Estimated completion: ~8s
```

**When to use:** Long workflows with distinct phases

### Level 3: Detailed (Verbose Mode)

Show every sub-operation:

```
Executing: complex-workflow.cheesecake [VERBOSE]

[■■■■■■■□□□] 70% complete

✓ Phase 1: Research          [DONE]
  ✓ Created Researcher agent
  ✓ Spawned 3 parallel sessions
  ✓ Collected results
  ✓ Saved checkpoint

→ Phase 2: Writing            [RUNNING]
  ✓ Created Writer agent
  → Running session (task: "Write article")
    Model: opus
    Estimated tokens: ~2000
  ○ Pending: Review loop
  ○ Pending: Save output

Tokens: 12,450 | Budget: 20,000 | Remaining: 7,550
Time: 45s elapsed, ~20s remaining
```

**When to use:** Debugging, monitoring expensive workflows

---

## Progress Display Format

### Visual Progress Bar

```
[■■■■■■■□□□] 70%
```

**Rules:**
- Always 10 segments
- ■ = completed
- □ = pending
- Percentage shown to right

### Status Indicators

- `✓` - Completed successfully
- `→` - Currently running
- `○` - Pending (not started)
- `⚠` - Warning occurred
- `✗` - Failed

### Phase States

| State | Display | Meaning |
|-------|---------|---------|
| PENDING | `○ Phase Name [PENDING]` | Not yet started |
| RUNNING | `→ Phase Name [RUNNING] 5.2s` | Currently executing |
| DONE | `✓ Phase Name [DONE] 5.2s` | Completed successfully |
| FAILED | `✗ Phase Name [FAILED]` | Error occurred |

---

## PHASE Construct (Optional)

Users can optionally mark execution phases for better progress tracking:

### Syntax

```cheesecake
PHASE "phase-name":
  # workflow code for this phase
END PHASE
```

### Example

```cheesecake
# Research and write workflow with phases

PHASE "Research":
  VAR researcher = NEW Researcher()
  VAR findings = RUN SESSION(researcher): TASK: "Research topic"
  CHECKPOINT "research-complete": SAVE: {findings}
END PHASE

PHASE "Analysis":
  VAR analyst = NEW Analyst()
  VAR insights = RUN SESSION(analyst): TASK: "Analyze findings"
END PHASE

PHASE "Writing":
  VAR writer = NEW Writer()
  VAR article = RUN SESSION(writer): TASK: "Write article"
END PHASE

PHASE "Output":
  SAVE article TO "output/article.md"
  LOG SUCCESS: "Article complete"
END PHASE
```

### Benefits of PHASE Blocks

1. **Clear progress** - Users see which phase is running
2. **Better estimates** - More accurate completion predictions
3. **Resumability** - Can checkpoint between phases
4. **Debugging** - Easier to identify where issues occur

### PHASE Block Rules

- Phase names must be unique within a workflow
- Phases execute sequentially (not parallel)
- PARALLEL blocks can exist inside phases
- Phases are purely organizational (no functional change)
- Workflows without PHASE blocks still show progress

---

## Progress Calculation

### Statement-Level Progress

```
progress = (completed_statements / total_statements) * 100
```

**Example:**
```cheesecake
VAR x = 1          # Statement 1
VAR y = 2          # Statement 2
VAR z = RUN ...    # Statement 3 (currently running)
PRINT z            # Statement 4
```

When executing statement 3:
- Total statements: 4
- Completed: 2
- Progress: (2/4) * 100 = 50%

### Phase-Level Progress

```
progress = (completed_phases / total_phases) * 100
```

**Example:**
```cheesecake
PHASE "A": ... END PHASE  # Phase 1 (done)
PHASE "B": ... END PHASE  # Phase 2 (done)
PHASE "C": ... END PHASE  # Phase 3 (running)
PHASE "D": ... END PHASE  # Phase 4 (pending)
```

When executing Phase 3:
- Total phases: 4
- Completed: 2
- Progress: (2/4) * 100 = 50%

### Weighted Progress (Advanced)

For more accurate progress with nested structures:

```
progress = Σ(statement_weight * completion) / total_weight
```

**Statement weights:**
- Simple assignment: 1 unit
- SESSION execution: 10 units
- PARALLEL block: 5 units per session
- LOOP: iterations * inner_weight

**This provides more realistic progress** - a SESSION that spawns an agent should count more than a simple variable assignment.

---

## Token Tracking

### Token Counting

Track tokens for each operation:

| Operation | Estimated Tokens |
|-----------|------------------|
| Sonnet session (simple task) | ~500-1000 |
| Sonnet session (complex task) | ~2000-5000 |
| Opus session (simple task) | ~1000-2000 |
| Opus session (complex task) | ~3000-8000 |
| PARALLEL (N sessions) | Sum of all sessions |

### Display Format

```
Tokens: 12,450 used | ~8,000 remaining | Budget: 20,000
```

### Token Budget Warning

When approaching token budget:

```
⚠️ Token Warning: 18,500 / 20,000 used (92%)
   Estimated remaining operations: 2
   Consider increasing budget or reducing scope.
```

---

## Time Tracking

### Display Format

```
Time: 45s elapsed | ~20s remaining | Est. completion: 1m 5s
```

### Time Estimation

Based on:
1. **Average time per statement** from completed statements
2. **Known operation times** (SESSION ≈ 2-5s, PARALLEL ≈ 3-8s)
3. **Historical data** if available

**Formula:**
```
remaining_time = (remaining_statements * avg_time_per_statement)
```

---

## Progress Update Frequency

### Real-time Updates

Update progress display:
- **After each statement completes**
- **Every 2-3 seconds during long operations**
- **When entering/exiting phases**
- **On significant state changes** (checkpoint, error, etc.)

### Buffering

To avoid screen flicker:
- Buffer updates if less than 1 second apart
- Always show final completion status

---

## Progress Display Examples

### Example 1: Simple Workflow (No Phases)

```
╔═══════════════════════════════════════════════════════════╗
║  Executing: hello-world.cheesecake                        ║
╚═══════════════════════════════════════════════════════════╝

[■■■■■□□□□□] 50% complete (2/4 statements)

Current: Running SESSION (line 20)
  Task: "Say hello to someone learning CheeseCake..."
  Model: sonnet
  Est. tokens: ~800

Elapsed: 3.2s
Tokens: 425 used

Status: Running...
```

### Example 2: Multi-Phase Workflow

```
╔═══════════════════════════════════════════════════════════╗
║  Executing: research-workflow.cheesecake                  ║
╚═══════════════════════════════════════════════════════════╝

[■■■■■■□□□□] 60% complete

✓ Phase 1: Research           [DONE]     8.5s
  • Created Researcher agent
  • 3 parallel sessions completed
  • Checkpoint saved

✓ Phase 2: Analysis           [DONE]     4.2s
  • Analysis complete

→ Phase 3: Writing            [RUNNING]  2.1s
  • Draft generation in progress...

○ Phase 4: Output             [PENDING]

─────────────────────────────────────────────────────────────
Tokens: 8,420 used | ~4,000 remaining
Time: 14.8s elapsed | ~7s remaining
─────────────────────────────────────────────────────────────
```

### Example 3: Completion

```
╔═══════════════════════════════════════════════════════════╗
║  Execution Complete! ✓                                    ║
╚═══════════════════════════════════════════════════════════╝

[■■■■■■■■■■] 100% complete

✓ Phase 1: Research           [DONE]     8.5s
✓ Phase 2: Analysis           [DONE]     4.2s
✓ Phase 3: Writing            [DONE]     9.3s
✓ Phase 4: Output             [DONE]     0.8s

─────────────────────────────────────────────────────────────
Total time: 22.8s
Tokens used: 12,420
Output: output/article.md

Status: SUCCESS
─────────────────────────────────────────────────────────────
```

---

## Error Handling

### When Errors Occur

```
╔═══════════════════════════════════════════════════════════╗
║  Execution Failed ✗                                       ║
╚═══════════════════════════════════════════════════════════╝

[■■■■■■□□□□] 60% complete (stopped)

✓ Phase 1: Research           [DONE]
✓ Phase 2: Analysis           [DONE]
✗ Phase 3: Writing            [FAILED]

Error at line 45: Session timeout after 30s

─────────────────────────────────────────────────────────────
Tokens used: 8,420
Time elapsed: 42.5s

Last checkpoint: "analysis-complete" (line 38)
You can resume from this checkpoint.
─────────────────────────────────────────────────────────────
```

---

## VM Implementation Guidelines

### When Executing a Workflow

1. **Parse the file** - Count total statements/phases
2. **Initialize progress tracker:**
   ```
   progress = {
     total: statement_count,
     completed: 0,
     current: null,
     start_time: NOW(),
     tokens_used: 0
   }
   ```

3. **Before each statement:**
   ```
   progress.current = statement_description
   DISPLAY progress_bar()
   ```

4. **After each statement:**
   ```
   progress.completed += 1
   progress.tokens_used += statement_tokens
   DISPLAY progress_bar()
   ```

5. **On completion:**
   ```
   DISPLAY completion_summary()
   ```

### Progress Display Function

```javascript
function display_progress(progress):
  bar = create_bar(progress.completed / progress.total)
  percentage = (progress.completed / progress.total) * 100

  print("[{bar}] {percentage}% complete")
  print("Current: {progress.current}")
  print("Elapsed: {elapsed_time}s")
  print("Tokens: {progress.tokens_used}")
```

---

## Configuration Options

Users can configure progress display via CONFIG block:

```cheesecake
CONFIG:
  PROGRESS_MODE: "detailed"     # "simple", "detailed", "verbose", "off"
  SHOW_TOKENS: true             # Show token tracking
  SHOW_TIME: true               # Show time estimates
  UPDATE_FREQUENCY: 2s          # How often to update
END CONFIG
```

---

## Best Practices

### For Users

1. **Use PHASE blocks** for long workflows (>5 minutes)
2. **Name phases descriptively** - "Research" not "Phase 1"
3. **Keep phases focused** - Each should have a clear purpose
4. **Add checkpoints** between phases for resumability

### For VM Implementation

1. **Update progress after every statement**
2. **Use weighted progress** for accuracy
3. **Handle nested structures** (loops, parallel) correctly
4. **Display errors clearly** with recovery options
5. **Don't block on progress updates** - async display

---

## Summary

Progress tracking transforms CheeseCake from a "black box" to a transparent, observable system:

✅ **Users see what's happening** in real-time
✅ **Token usage is visible** - no surprises
✅ **Time estimates help planning** - know how long workflows take
✅ **PHASE blocks organize** complex workflows
✅ **Errors show context** - easier debugging

This makes CheeseCake professional-grade and production-ready! 🚀
