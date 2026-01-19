# /cheesecake replay
# Purpose: Replay a previous execution
# Part of: CheeseCake v0.0.2 - Module 14 (History & Replay)
#
# This command replays a previous execution with the same or modified inputs,
# optionally resuming from a checkpoint.
#
# Usage:
#   /cheesecake replay #N              - Replay execution #N
#   /cheesecake replay #N --modify     - Replay with modified inputs
#   /cheesecake replay #N --from CP    - Resume from checkpoint
#
# Dependencies:
#   - skills/cheesecake/history.md
#   - skills/cheesecake/vm.md
#   - commands/cheesecake-history.md
#
# Related:
#   - commands/cheesecake-run.md

---
description: Replay a previous CheeseCake execution
---

# Replay Command

Replay a previous execution with the same inputs, modified inputs, or resume from a checkpoint.

---

## Basic Usage

### Replay with Same Inputs

```
/cheesecake replay #1
```

**Output:**
```
┌─────────────────────────────────────────────────────────────────────────────┐
│  Replay Execution #1                                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Program:    pipeline.cheesecake                                            │
│  Original:   2026-01-14 14:20 (2m 45s, $0.12)                              │
│                                                                             │
│  Inputs from original execution:                                            │
│    topic:        "machine learning trends"                                 │
│    depth:        "deep"                                                    │
│    max_sources:  10                                                        │
│                                                                             │
│  Proceed with replay? [Y/n]: y                                             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

Running pipeline.cheesecake...

[■■■■■■□□□□] 60% - Analysis phase
...
```

---

## Replay with Modified Inputs

### Interactive Modification

```
/cheesecake replay #1 --modify
```

**Output:**
```
┌─────────────────────────────────────────────────────────────────────────────┐
│  Replay Execution #1 (Modified)                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Program:    pipeline.cheesecake                                            │
│                                                                             │
│  Original inputs (press Enter to keep, or type new value):                 │
│                                                                             │
│    topic ["machine learning trends"]: artificial intelligence              │
│    depth ["deep"]: <Enter>                                                 │
│    max_sources [10]: 20                                                    │
│                                                                             │
│  Modified inputs:                                                           │
│    topic:        "artificial intelligence"  (changed)                      │
│    depth:        "deep"                     (unchanged)                    │
│    max_sources:  20                         (changed)                      │
│                                                                             │
│  Proceed with replay? [Y/n]: y                                             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Direct Input Override

```
/cheesecake replay #1 --input topic="quantum computing" --input max_sources=5
```

Override specific inputs without interactive prompt.

### JSON Input Override

```
/cheesecake replay #1 --inputs '{"topic": "quantum computing", "depth": "shallow"}'
```

Override multiple inputs via JSON.

---

## Replay from Checkpoint

### Resume from Named Checkpoint

```
/cheesecake replay #1 --from research-complete
```

**Output:**
```
┌─────────────────────────────────────────────────────────────────────────────┐
│  Replay Execution #1 from Checkpoint                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Program:       pipeline.cheesecake                                         │
│  Checkpoint:    research-complete                                           │
│                                                                             │
│  Checkpoint state includes:                                                 │
│    raw_data:    [loaded from checkpoint]                                   │
│    sources:     ["arxiv.org/...", "nature.com/...", ...]                  │
│                                                                             │
│  Skipping phases:                                                           │
│    ○ Research    (using checkpoint state)                                  │
│                                                                             │
│  Will execute:                                                              │
│    → Analysis                                                              │
│    → Output                                                                │
│                                                                             │
│  Estimated:                                                                 │
│    Duration:   ~1m 30s (skipping Research phase)                           │
│    Cost:       ~$0.06                                                      │
│                                                                             │
│  Proceed? [Y/n]: y                                                         │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### List Available Checkpoints

```
/cheesecake replay #1 --list-checkpoints
```

**Output:**
```
Available checkpoints for execution #1:

  Checkpoint               Created              Variables Saved
  ──────────────────────── ──────────────────── ─────────────────────────────
  research-complete        2026-01-14 14:20:45  raw_data, sources
  analysis-complete        2026-01-14 14:21:45  findings, confidence

Use: /cheesecake replay #1 --from <checkpoint-name>
```

---

## Replay Options

### Skip Confirmation

```
/cheesecake replay #1 --yes
```

Skip the confirmation prompt and replay immediately.

### Dry-Run

```
/cheesecake replay #1 --dry-run
```

**Output:**
```
DRY RUN: Replay of execution #1

Would execute:
  Program:     pipeline.cheesecake
  Inputs:      topic="machine learning trends", depth="deep", max_sources=10

Estimated:
  Sessions:    5-8
  Duration:    2-3 minutes
  Cost:        $0.10-0.15

To actually replay, run without --dry-run.
```

### Force Replay of Failed Execution

```
/cheesecake replay #3 --force
```

Replay a failed execution even if it previously failed. Useful after fixing the issue that caused the failure.

### With Different Config

```
/cheesecake replay #1 --budget 0.50 --model opus
```

Override CONFIG settings for the replay.

---

## Replay Tracking

Each replay creates a new history entry with metadata linking to the original:

```json
{
  "id": "j1k2l3",
  "program": "pipeline.cheesecake",
  "metadata": {
    "triggered_by": "replay",
    "parent_execution": "a1b2c3",
    "replayed_from_checkpoint": null,
    "modified_inputs": []
  }
}
```

When replaying with modifications:

```json
{
  "id": "m4n5o6",
  "program": "pipeline.cheesecake",
  "metadata": {
    "triggered_by": "replay",
    "parent_execution": "a1b2c3",
    "replayed_from_checkpoint": null,
    "modified_inputs": ["topic", "max_sources"]
  }
}
```

When replaying from checkpoint:

```json
{
  "id": "p7q8r9",
  "program": "pipeline.cheesecake",
  "metadata": {
    "triggered_by": "replay",
    "parent_execution": "a1b2c3",
    "replayed_from_checkpoint": "research-complete",
    "modified_inputs": []
  }
}
```

---

## Replay Chain Visualization

```
/cheesecake history #1 --chain
```

**Output:**
```
Execution Chain for #1 (a1b2c3):

  #1 (a1b2c3) ─── Original execution
       │         2026-01-14 14:20 ✓
       │
       ├──→ #6 (j1k2l3) ─── Replay (same inputs)
       │         2026-01-14 16:00 ✓
       │
       ├──→ #8 (m4n5o6) ─── Replay (modified: topic)
       │         2026-01-15 09:30 ✓
       │
       └──→ #10 (p7q8r9) ─── Replay from checkpoint
                 2026-01-15 11:00 ✓
```

---

## Execution Behavior

When the `/cheesecake replay` command is invoked:

### 1. Load Original Execution

```
Load execution record from history
VALIDATE execution exists
VALIDATE program file still exists
```

### 2. Handle Checkpoint Replay

```
IF --from specified:
  VALIDATE checkpoint exists in execution
  Load checkpoint state
  Set resume_point to checkpoint
END IF
```

### 3. Handle Input Modification

```
IF --modify:
  Prompt user for each input
  Collect modified values
ELIF --input specified:
  Parse input overrides
  Apply to original inputs
END IF
```

### 4. Confirm and Execute

```
IF NOT --yes:
  Show replay summary
  Ask for confirmation
END IF

IF confirmed:
  Create new execution record with:
    triggered_by: "replay"
    parent_execution: original_id
    replayed_from_checkpoint: checkpoint_name (if any)
    modified_inputs: list of changed inputs

  IF checkpoint resume:
    Load checkpoint state
    Resume from checkpoint point
  ELSE:
    Execute from beginning with inputs
  END IF
END IF
```

### 5. Record Result

```
Complete execution record
Link to parent execution in metadata
Save to history
```

---

## Options Reference

| Option | Description | Example |
|--------|-------------|---------|
| `#N` | Execution number to replay | `#1`, `#42` |
| `--modify` | Interactive input modification | |
| `--input K=V` | Override specific input | `--input topic="AI"` |
| `--inputs JSON` | Override inputs via JSON | `--inputs '{"topic":"AI"}'` |
| `--from CP` | Resume from checkpoint | `--from research-complete` |
| `--list-checkpoints` | List available checkpoints | |
| `--yes` `-y` | Skip confirmation | |
| `--dry-run` | Preview without executing | |
| `--force` | Replay failed execution | |
| `--budget N` | Override budget config | `--budget 0.50` |
| `--model M` | Override default model | `--model opus` |

---

## Error Messages

### Execution Not Found

```
Execution #99 not found.

Use /cheesecake history to see available executions.
```

### Program File Missing

```
Cannot replay execution #1.

The program file no longer exists:
  pipeline.cheesecake

The original execution was:
  2026-01-14 14:20
  Status: Success
```

### Checkpoint Not Found

```
Checkpoint 'invalid-checkpoint' not found in execution #1.

Available checkpoints:
  - research-complete
  - analysis-complete

Use: /cheesecake replay #1 --list-checkpoints
```

### Checkpoint State Corrupted

```
Cannot load checkpoint 'research-complete'.

The checkpoint state file may be corrupted or deleted.

Options:
  1. Replay from beginning: /cheesecake replay #1
  2. Try a different checkpoint: /cheesecake replay #1 --from analysis-complete
```

### Invalid Input Override

```
Invalid input override: 'topic'

The original execution did not have an input named 'topic'.

Original inputs:
  - subject
  - depth
  - max_results
```

---

## Related Commands

| Command | Purpose |
|---------|---------|
| `/cheesecake history` | View execution history |
| `/cheesecake history #N` | View execution details |
| `/cheesecake run` | Execute a CheeseCake program |
