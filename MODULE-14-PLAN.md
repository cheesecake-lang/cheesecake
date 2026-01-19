# Module 14: History & Replay

**Module**: History & Replay (v0.0.2)
**Status**: In Progress
**Date**: 2026-01-14

---

## Overview

Module 14 adds execution history tracking and replay capabilities to CheeseCake. This enables developers to:
- Track past executions of their workflows
- View detailed execution logs
- Replay previous executions with same or modified inputs
- Debug failed executions by examining history
- Compare execution costs and performance over time

---

## Features

### 1. Execution History Tracking

Every execution of a `.cheesecake` program is automatically logged:

```
.cheesecake/
  history/
    2026-01-14T10-30-00_workflow_abc123.json
    2026-01-14T11-45-00_research_def456.json
    2026-01-14T14-20-00_pipeline_ghi789.json
```

**Execution Record Format:**

```json
{
  "id": "abc123",
  "program": "workflow.cheesecake",
  "started_at": "2026-01-14T10:30:00Z",
  "completed_at": "2026-01-14T10:32:45Z",
  "duration_seconds": 165,
  "status": "success",
  "inputs": {
    "topic": "machine learning",
    "depth": "deep"
  },
  "outputs": {
    "findings": "...",
    "sources": ["..."]
  },
  "cost": {
    "total": 0.12,
    "sessions": 5,
    "tokens": {
      "input": 4500,
      "output": 2300
    }
  },
  "phases": [
    {"name": "Research", "duration": 45, "status": "success"},
    {"name": "Analysis", "duration": 60, "status": "success"},
    {"name": "Output", "duration": 60, "status": "success"}
  ],
  "checkpoints": ["research-complete", "analysis-complete"],
  "errors": [],
  "config": {
    "budget": 1.00,
    "model_default": "sonnet"
  }
}
```

### 2. History Commands

#### List History
```
/cheesecake history

Recent executions:
  #1  2026-01-14 14:20  pipeline.cheesecake       ✓ Success   2m 45s   $0.12
  #2  2026-01-14 11:45  research.cheesecake       ✓ Success   1m 30s   $0.08
  #3  2026-01-14 10:30  workflow.cheesecake       ✗ Failed    0m 45s   $0.03
  #4  2026-01-13 16:00  data-processor.cheesecake ✓ Success   5m 12s   $0.25

Showing 4 of 4 executions. Use --all to see more.
```

#### View Execution Details
```
/cheesecake history #1

Execution #1: pipeline.cheesecake
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Status:     ✓ Success
Started:    2026-01-14 14:20:00
Duration:   2m 45s
Cost:       $0.12 (5 sessions, 6,800 tokens)

Inputs:
  topic: "machine learning"
  depth: "deep"

Outputs:
  findings: "Machine learning trends show..."
  sources: ["arxiv.org/...", "nature.com/..."]

Phases:
  ✓ Research    [45s]   $0.04
  ✓ Analysis    [60s]   $0.05
  ✓ Output      [60s]   $0.03

Checkpoints:
  ✓ research-complete
  ✓ analysis-complete

Commands:
  /cheesecake replay #1           Replay with same inputs
  /cheesecake replay #1 --modify  Replay with modified inputs
```

#### Replay Execution
```
/cheesecake replay #1

Replaying execution #1: pipeline.cheesecake
Using inputs from previous execution:
  topic: "machine learning"
  depth: "deep"

Proceed? [Y/n]
```

```
/cheesecake replay #1 --modify

Replaying execution #1: pipeline.cheesecake

Previous inputs:
  topic: "machine learning"
  depth: "deep"

Modify inputs? (Enter new values or press Enter to keep)
  topic ["machine learning"]: artificial intelligence
  depth ["deep"]: <Enter>

Proceeding with:
  topic: "artificial intelligence"
  depth: "deep"
```

### 3. History Configuration

```cheesecake
CONFIG:
  # History settings
  HISTORY_ENABLED: TRUE           # Enable/disable history tracking
  HISTORY_RETENTION_DAYS: 30      # How long to keep history
  HISTORY_MAX_ENTRIES: 100        # Maximum history entries
  HISTORY_INCLUDE_OUTPUTS: TRUE   # Include full outputs in history
END CONFIG
```

### 4. Programmatic History Access

```cheesecake
# Get recent history
VAR recent = GET_HISTORY(limit: 5)
FOR execution IN recent:
  PRINT "#{execution.id}: {execution.program} - {execution.status}"
END FOR

# Get specific execution
VAR last_run = GET_EXECUTION(id: "abc123")
IF last_run.status == "success":
  PRINT "Last run succeeded with cost: ${last_run.cost.total}"
END IF

# Compare executions
VAR exec1 = GET_EXECUTION(id: "abc123")
VAR exec2 = GET_EXECUTION(id: "def456")
IF exec2.cost.total < exec1.cost.total:
  PRINT "Newer execution was cheaper!"
END IF
```

### 5. Replay from Checkpoint

```cheesecake
# Replay from a specific checkpoint
/cheesecake replay #1 --from-checkpoint research-complete

# This skips phases before the checkpoint and resumes from there
```

### 6. History Filtering

```
/cheesecake history --status failed     # Only failed executions
/cheesecake history --program workflow  # Only workflow.cheesecake
/cheesecake history --since 2026-01-01  # Since a date
/cheesecake history --cost-above 0.50   # Expensive executions
/cheesecake history --json              # Output as JSON
```

### 7. History Cleanup

```
/cheesecake history --clear             # Clear all history
/cheesecake history --clear --before 2026-01-01  # Clear old history
/cheesecake history --clear --failed    # Clear only failed executions
```

---

## Implementation Plan

### Files to Create

| File | Purpose | Lines (est.) |
|------|---------|--------------|
| `skills/cheesecake/history.md` | Complete history specification | 600+ |
| `commands/cheesecake-history.md` | History command | 300+ |
| `commands/cheesecake-replay.md` | Replay command | 200+ |
| `test-history.cheesecake` | Test file | 300+ |

### Files to Update

| File | Changes |
|------|---------|
| `skills/cheesecake/SKILL.md` | Add history section |
| `skills/cheesecake/vm.md` | Add history tracking semantics |
| `CHANGELOG.md` | Document Module 14 |

---

## VM Behavior

### Automatic History Recording

When VM executes a program:

```
1. BEFORE EXECUTION:
   - Generate unique execution ID
   - Record start timestamp
   - Capture inputs (from INPUT declarations or command line)
   - Initialize history record

2. DURING EXECUTION:
   - Track phase transitions
   - Record checkpoint creations
   - Accumulate cost data
   - Capture errors (if any)

3. AFTER EXECUTION:
   - Record completion timestamp
   - Calculate duration
   - Capture outputs (from OUTPUT declarations)
   - Finalize cost totals
   - Write history record to .cheesecake/history/
```

### History Storage Format

```
Filename: {timestamp}_{program}_{id}.json
Example:  2026-01-14T10-30-00_workflow_abc123.json

Location: .cheesecake/history/
```

### Replay Execution

When replaying:

```
1. Load execution record from history
2. Extract inputs
3. Optionally allow user to modify inputs
4. If --from-checkpoint specified:
   - Load checkpoint state
   - Resume from that point
5. Execute program with inputs
6. Record NEW execution in history (separate entry)
```

---

## Integration with Existing Features

### With CHECKPOINT
- History records which checkpoints were created
- Replay can resume from any recorded checkpoint
- Checkpoint state is preserved for replay

### With Cost Tracking (Module 11)
- History includes full cost breakdown
- Can filter by cost
- Compare costs across executions

### With Progress Tracking (Module 9)
- Phase information captured in history
- Duration per phase recorded
- Progress reconstruction possible

### With CONFIG
- History settings in CONFIG block
- Retention and limits configurable
- Can disable history entirely

### With Program Contracts
- Inputs captured from INPUT declarations
- Outputs captured from OUTPUT declarations
- Replay uses captured inputs

---

## Test Scenarios

1. **Basic History Recording** - Execute program, verify history created
2. **History Listing** - Multiple executions, list shows all
3. **Execution Details** - View details of specific execution
4. **Replay Same Inputs** - Replay and verify same inputs used
5. **Replay Modified Inputs** - Modify inputs during replay
6. **Replay from Checkpoint** - Resume from checkpoint
7. **History Filtering** - Filter by status, program, date, cost
8. **History Cleanup** - Clear history with various options
9. **Programmatic Access** - GET_HISTORY, GET_EXECUTION functions
10. **Failed Execution History** - Error info captured correctly
11. **Cost Tracking Integration** - Cost data accurate in history
12. **History Retention** - Old entries cleaned up per config
13. **History Disabled** - HISTORY_ENABLED: FALSE works
14. **Large Output Handling** - Outputs truncated if too large
15. **Concurrent Executions** - Multiple simultaneous runs tracked

---

## Success Criteria

- [ ] History records created automatically for each execution
- [ ] `/cheesecake history` lists past executions
- [ ] `/cheesecake history #N` shows execution details
- [ ] `/cheesecake replay #N` replays with same inputs
- [ ] `/cheesecake replay #N --modify` allows input changes
- [ ] History filtering works (status, program, date, cost)
- [ ] History cleanup works
- [ ] CONFIG settings control history behavior
- [ ] GET_HISTORY and GET_EXECUTION functions work
- [ ] Integration with checkpoints works
- [ ] Integration with cost tracking works
- [ ] All 15 test scenarios pass

---

## Notes

- History files use JSON for easy parsing and debugging
- Execution IDs are short hashes for readability
- Large outputs may be truncated in history (configurable)
- History is local to the project (not shared across machines)
- Replay creates a NEW history entry (doesn't modify original)
