# CheeseCake History & Replay Specification

## Overview

The History & Replay system automatically tracks all executions of `.cheesecake` programs and enables replaying past executions. This provides:

- **Audit Trail**: Complete record of what was executed, when, and with what results
- **Debugging**: Examine failed executions to understand what went wrong
- **Reproducibility**: Replay executions with identical or modified inputs
- **Cost Analysis**: Track and compare costs across executions
- **Performance Monitoring**: Analyze execution times and identify bottlenecks

---

## 1. Execution History

### Automatic Recording

Every execution of a `.cheesecake` program is automatically recorded unless explicitly disabled:

```cheesecake
CONFIG:
  HISTORY_ENABLED: FALSE  # Disable history for this program
END CONFIG
```

By default, `HISTORY_ENABLED: TRUE`.

### History Storage Location

History is stored in the project's `.cheesecake/history/` directory:

```
.cheesecake/
  history/
    2026-01-14T10-30-00_workflow_a1b2c3.json
    2026-01-14T11-45-00_research_d4e5f6.json
    2026-01-14T14-20-00_pipeline_g7h8i9.json
    index.json  # Quick lookup index
```

### Execution Record Structure

Each execution creates a JSON record:

```json
{
  "id": "a1b2c3",
  "program": "workflow.cheesecake",
  "program_path": "/path/to/workflow.cheesecake",
  "version": "1.0.0",

  "timing": {
    "started_at": "2026-01-14T10:30:00Z",
    "completed_at": "2026-01-14T10:32:45Z",
    "duration_ms": 165000
  },

  "status": "success",
  "exit_code": 0,

  "inputs": {
    "topic": "machine learning trends",
    "depth": "deep",
    "max_sources": 10
  },

  "outputs": {
    "findings": "Analysis of ML trends shows...",
    "sources": ["arxiv.org/paper1", "nature.com/article2"],
    "confidence": 0.92
  },

  "cost": {
    "total_usd": 0.12,
    "sessions_count": 5,
    "tokens": {
      "input": 4500,
      "output": 2300,
      "total": 6800
    },
    "by_model": {
      "sonnet": {"sessions": 4, "cost": 0.08},
      "opus": {"sessions": 1, "cost": 0.04}
    }
  },

  "phases": [
    {
      "name": "Research",
      "started_at": "2026-01-14T10:30:00Z",
      "completed_at": "2026-01-14T10:30:45Z",
      "duration_ms": 45000,
      "status": "success",
      "sessions": 2,
      "cost": 0.04
    },
    {
      "name": "Analysis",
      "started_at": "2026-01-14T10:30:45Z",
      "completed_at": "2026-01-14T10:31:45Z",
      "duration_ms": 60000,
      "status": "success",
      "sessions": 2,
      "cost": 0.05
    },
    {
      "name": "Output",
      "started_at": "2026-01-14T10:31:45Z",
      "completed_at": "2026-01-14T10:32:45Z",
      "duration_ms": 60000,
      "status": "success",
      "sessions": 1,
      "cost": 0.03
    }
  ],

  "checkpoints": [
    {
      "name": "research-complete",
      "created_at": "2026-01-14T10:30:45Z",
      "variables_saved": ["raw_data", "sources"]
    },
    {
      "name": "analysis-complete",
      "created_at": "2026-01-14T10:31:45Z",
      "variables_saved": ["findings", "confidence"]
    }
  ],

  "errors": [],

  "config": {
    "budget": 1.00,
    "default_model": "sonnet",
    "history_enabled": true
  },

  "environment": {
    "cheesecake_version": "0.0.2",
    "platform": "darwin",
    "working_directory": "/path/to/project"
  },

  "metadata": {
    "triggered_by": "cli",
    "parent_execution": null,
    "tags": ["research", "ml"]
  }
}
```

### Failed Execution Records

Failed executions include error details:

```json
{
  "id": "x1y2z3",
  "program": "risky-workflow.cheesecake",
  "status": "failed",
  "exit_code": 1,

  "error": {
    "type": "SessionError",
    "message": "API rate limit exceeded",
    "phase": "Research",
    "line": 45,
    "statement": "VAR data = RUN SESSION(researcher): TASK: \"Fetch data\"",
    "stack": [
      "at Research phase, line 45",
      "at PARALLEL block, line 40",
      "at main execution"
    ]
  },

  "partial_outputs": {
    "sources": ["partial-source-1"]
  },

  "phases": [
    {"name": "Setup", "status": "success", "duration_ms": 5000},
    {"name": "Research", "status": "failed", "duration_ms": 30000}
  ]
}
```

---

## 2. History Index

For fast lookups, an index file maintains summary information:

```json
{
  "last_updated": "2026-01-14T14:20:00Z",
  "total_executions": 47,
  "executions": [
    {
      "id": "a1b2c3",
      "program": "workflow.cheesecake",
      "status": "success",
      "started_at": "2026-01-14T10:30:00Z",
      "duration_ms": 165000,
      "cost_usd": 0.12
    },
    {
      "id": "d4e5f6",
      "program": "research.cheesecake",
      "status": "success",
      "started_at": "2026-01-14T11:45:00Z",
      "duration_ms": 90000,
      "cost_usd": 0.08
    }
  ]
}
```

---

## 3. History Configuration

Configure history behavior in the CONFIG block:

```cheesecake
CONFIG:
  # Enable/disable history
  HISTORY_ENABLED: TRUE

  # Retention settings
  HISTORY_RETENTION_DAYS: 30        # Delete entries older than 30 days
  HISTORY_MAX_ENTRIES: 100          # Keep at most 100 entries

  # Content settings
  HISTORY_INCLUDE_OUTPUTS: TRUE     # Include full outputs
  HISTORY_OUTPUT_MAX_SIZE: 10000    # Truncate outputs larger than 10KB
  HISTORY_INCLUDE_INPUTS: TRUE      # Include inputs

  # Privacy settings
  HISTORY_REDACT_SECRETS: TRUE      # Redact values matching secret patterns
END CONFIG
```

### Configuration Defaults

| Setting | Default | Description |
|---------|---------|-------------|
| `HISTORY_ENABLED` | `TRUE` | Enable history tracking |
| `HISTORY_RETENTION_DAYS` | `30` | Days to keep history |
| `HISTORY_MAX_ENTRIES` | `100` | Maximum history entries |
| `HISTORY_INCLUDE_OUTPUTS` | `TRUE` | Store outputs in history |
| `HISTORY_OUTPUT_MAX_SIZE` | `10000` | Max output size in bytes |
| `HISTORY_INCLUDE_INPUTS` | `TRUE` | Store inputs in history |
| `HISTORY_REDACT_SECRETS` | `TRUE` | Redact secret values |

### Secret Redaction

When `HISTORY_REDACT_SECRETS: TRUE`, values matching these patterns are redacted:

- Input/output names containing: `password`, `secret`, `key`, `token`, `credential`
- Values that look like API keys (long alphanumeric strings)

```json
{
  "inputs": {
    "api_key": "[REDACTED]",
    "topic": "machine learning"
  }
}
```

---

## 4. Programmatic History Access

### GET_HISTORY Function

Retrieve recent execution history:

```cheesecake
# Get last 5 executions
VAR recent = GET_HISTORY(limit: 5)

# Get executions with filters
VAR failed = GET_HISTORY(
  status: "failed",
  limit: 10
)

VAR expensive = GET_HISTORY(
  cost_above: 0.50,
  limit: 5
)

VAR recent_workflow = GET_HISTORY(
  program: "workflow.cheesecake",
  since: "2026-01-01",
  limit: 10
)
```

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `limit` | Number | Maximum entries to return (default: 10) |
| `status` | String | Filter by status: "success", "failed", "all" |
| `program` | String | Filter by program name |
| `since` | String | Only entries after this date (ISO format) |
| `before` | String | Only entries before this date |
| `cost_above` | Number | Only entries with cost above this |
| `cost_below` | Number | Only entries with cost below this |

**Returns:** Array of execution summary objects

```cheesecake
VAR history = GET_HISTORY(limit: 3)

FOR exec IN history:
  PRINT "#{exec.id}: {exec.program}"
  PRINT "  Status: {exec.status}"
  PRINT "  Duration: {exec.duration_ms}ms"
  PRINT "  Cost: ${exec.cost_usd}"
END FOR
```

### GET_EXECUTION Function

Retrieve full details of a specific execution:

```cheesecake
# By ID
VAR exec = GET_EXECUTION(id: "a1b2c3")

# Most recent execution
VAR last = GET_EXECUTION(latest: TRUE)

# Most recent of specific program
VAR last_workflow = GET_EXECUTION(
  program: "workflow.cheesecake",
  latest: TRUE
)
```

**Returns:** Full execution record object (or NULL if not found)

```cheesecake
VAR exec = GET_EXECUTION(id: "a1b2c3")

IF exec IS NOT NULL:
  PRINT "Program: {exec.program}"
  PRINT "Started: {exec.timing.started_at}"
  PRINT "Duration: {exec.timing.duration_ms}ms"
  PRINT "Status: {exec.status}"

  IF exec.status == "failed":
    PRINT "Error: {exec.error.message}"
    PRINT "At line: {exec.error.line}"
  END IF

  PRINT "Inputs:"
  FOR key, value IN exec.inputs:
    PRINT "  {key}: {value}"
  END FOR

  PRINT "Cost breakdown:"
  FOR model, data IN exec.cost.by_model:
    PRINT "  {model}: ${data.cost} ({data.sessions} sessions)"
  END FOR
END IF
```

### COMPARE_EXECUTIONS Function

Compare two executions:

```cheesecake
VAR comparison = COMPARE_EXECUTIONS(
  id1: "a1b2c3",
  id2: "d4e5f6"
)

PRINT "Cost difference: ${comparison.cost_diff}"
PRINT "Duration difference: {comparison.duration_diff}ms"
PRINT "Same inputs: {comparison.inputs_match}"
PRINT "Same outputs: {comparison.outputs_match}"
```

---

## 5. Replay Functionality

### Basic Replay

Replay an execution with the same inputs:

```cheesecake
# Replay by execution ID
REPLAY execution_id: "a1b2c3"

# This is equivalent to running the program with captured inputs:
# RUN "workflow.cheesecake" WITH {
#   topic: "machine learning trends",
#   depth: "deep"
# }
```

### Replay with Modified Inputs

```cheesecake
# Get previous execution
VAR prev = GET_EXECUTION(id: "a1b2c3")

# Modify inputs
VAR new_inputs = prev.inputs
new_inputs.topic = "artificial intelligence"
new_inputs.depth = "shallow"

# Replay with modified inputs
REPLAY execution_id: "a1b2c3" WITH new_inputs
```

### Replay from Checkpoint

```cheesecake
# Replay starting from a specific checkpoint
REPLAY execution_id: "a1b2c3" FROM_CHECKPOINT: "analysis-complete"

# This:
# 1. Loads the checkpoint state from the original execution
# 2. Resumes execution from that point
# 3. Creates a new history entry
```

### Replay Behavior

When replaying:

1. **New Execution ID**: Each replay creates a new history entry
2. **Parent Reference**: The new entry references the original execution
3. **Checkpoint State**: If using FROM_CHECKPOINT, loads saved state
4. **Modified Tracking**: Records which inputs were modified (if any)

```json
{
  "id": "j1k2l3",
  "program": "workflow.cheesecake",
  "metadata": {
    "triggered_by": "replay",
    "parent_execution": "a1b2c3",
    "replayed_from_checkpoint": "analysis-complete",
    "modified_inputs": ["topic"]
  }
}
```

---

## 6. History Cleanup

### Automatic Cleanup

Based on CONFIG settings, old entries are automatically cleaned up:

```cheesecake
CONFIG:
  HISTORY_RETENTION_DAYS: 30
  HISTORY_MAX_ENTRIES: 100
END CONFIG

# Entries older than 30 days are deleted
# If more than 100 entries exist, oldest are deleted
```

### Manual Cleanup

```cheesecake
# Clear all history
CLEAR_HISTORY()

# Clear with filters
CLEAR_HISTORY(before: "2026-01-01")
CLEAR_HISTORY(status: "failed")
CLEAR_HISTORY(program: "test.cheesecake")
```

---

## 7. History Events

History system emits events that can be handled:

```cheesecake
ON EVENT execution_recorded(record):
  LOG INFO: "Recorded execution {record.id}"

  IF record.status == "failed":
    # Notify about failures
    EMIT alert(
      type: "execution_failed",
      program: record.program,
      error: record.error.message
    )
  END IF
END ON

ON EVENT history_cleanup(deleted_count):
  LOG INFO: "Cleaned up {deleted_count} old history entries"
END ON
```

---

## 8. Best Practices

### 1. Use Tags for Organization

```cheesecake
CONFIG:
  HISTORY_TAGS: ["production", "daily-report"]
END CONFIG
```

Then filter by tags:
```cheesecake
VAR prod_runs = GET_HISTORY(tags: ["production"])
```

### 2. Capture Meaningful Outputs

```cheesecake
# Instead of just the final result
OUTPUT result = final_output

# Also capture metrics for history
OUTPUT metrics = {
  items_processed: count,
  success_rate: successes / total,
  average_confidence: avg_conf
}
```

### 3. Use Checkpoints for Long Workflows

```cheesecake
PHASE "Data Collection":
  VAR data = RUN SESSION(collector): TASK: "Collect data"
  CHECKPOINT "data-collected":
    SAVE: {data}
  END CHECKPOINT
END PHASE

# If something fails later, can replay from "data-collected"
```

### 4. Monitor Cost Trends

```cheesecake
# Get cost trend for a workflow
VAR history = GET_HISTORY(
  program: "daily-report.cheesecake",
  limit: 30
)

VAR total_cost = 0
FOR exec IN history:
  total_cost = total_cost + exec.cost_usd
END FOR

VAR avg_cost = total_cost / LENGTH(history)
LOG INFO: "Average cost over 30 runs: ${avg_cost}"
```

### 5. Debug Failed Executions

```cheesecake
# Find recent failures
VAR failures = GET_HISTORY(status: "failed", limit: 5)

FOR failure IN failures:
  VAR full = GET_EXECUTION(id: failure.id)
  PRINT "Failed: {full.program} at line {full.error.line}"
  PRINT "Error: {full.error.message}"
  PRINT "Inputs: {full.inputs}"
  PRINT "---"
END FOR
```

---

## 9. Privacy and Security

### Sensitive Data Handling

1. **Redaction**: Enable `HISTORY_REDACT_SECRETS` to automatically redact sensitive values
2. **Exclusion**: Use `HISTORY_EXCLUDE_INPUTS: ["password", "api_key"]` to exclude specific inputs
3. **Encryption**: History files can be encrypted (future feature)

### Access Control

History is stored locally in `.cheesecake/history/`. Consider:

1. Adding `.cheesecake/history/` to `.gitignore`
2. Setting appropriate file permissions
3. Clearing history before sharing project

---

## 10. Integration Summary

| Feature | Integration |
|---------|-------------|
| **CHECKPOINT** | Recorded in history, enables replay from checkpoint |
| **Cost Tracking** | Full cost breakdown in history records |
| **Progress/PHASE** | Phase timing and status recorded |
| **CONFIG** | History settings configurable |
| **Events** | execution_recorded, history_cleanup events |
| **Program Contracts** | INPUT/OUTPUT values captured |

---

## 11. Implementation Notes

### For the VM

When executing a program:

1. **Start Recording**
   ```
   execution_record = {
     id: generate_short_id(),
     program: current_program_name,
     started_at: NOW(),
     status: "running"
   }
   ```

2. **During Execution**
   - Capture inputs from INPUT declarations
   - Track phase transitions
   - Record checkpoint creations
   - Accumulate cost data
   - Capture any errors

3. **On Completion**
   ```
   execution_record.completed_at = NOW()
   execution_record.status = "success" OR "failed"
   execution_record.outputs = collect_outputs()
   write_to_history(execution_record)
   update_index()
   ```

4. **Cleanup Check**
   ```
   IF history_count > HISTORY_MAX_ENTRIES:
     delete_oldest_entries()
   IF any_entries_older_than(HISTORY_RETENTION_DAYS):
     delete_old_entries()
   ```

### ID Generation

Execution IDs are 6-character alphanumeric hashes for readability:

```
a1b2c3, x7y8z9, etc.
```

Generated from: `hash(program + timestamp + random)[:6]`
