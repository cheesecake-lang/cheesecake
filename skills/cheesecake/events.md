# CheeseCake Events & Scheduling Specification
# Purpose: Define event handling and scheduling constructs for reactive workflows
# Part of: CheeseCake v0.0.2 - Module 12 (Events & Scheduling)
#
# This file specifies how events and schedules work in CheeseCake.
# Use this to understand and implement reactive workflow patterns.
#
# Usage:
#   Referenced when executing workflows with ON EVENT, SCHEDULE, EMIT, or LISTEN
#
# Dependencies:
#   - SKILL.md (language specification)
#   - vm.md (execution semantics)
#
# Related:
#   - interactive.md (user-triggered events)
#   - cost-management.md (event cost tracking)

---

# Events & Scheduling in CheeseCake

## 1. Overview

### Purpose

CheeseCake's event system enables **reactive programming patterns** where workflows respond to events rather than just executing sequentially. This includes:

1. **Event Handlers**: Respond to external or internal triggers
2. **Scheduled Tasks**: Execute at specific times or intervals
3. **Event Emission**: Trigger custom events within workflows
4. **Event Listening**: Coordinate between workflow components

### Core Concepts

```
┌─────────────────────────────────────────────────────────────────┐
│                    CheeseCake Event System                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  External Events          Internal Events        Scheduled      │
│  ┌─────────────┐         ┌─────────────┐        ┌──────────┐   │
│  │ file_changed│         │ EMIT custom │        │ INTERVAL │   │
│  │ api_webhook │    →    │ events      │   ←    │ CRON     │   │
│  │ user_input  │         │             │        │ ONCE_AT  │   │
│  └──────┬──────┘         └──────┬──────┘        └────┬─────┘   │
│         │                       │                    │          │
│         └───────────────────────┼────────────────────┘          │
│                                 ▼                               │
│                     ┌─────────────────────┐                     │
│                     │    Event Registry   │                     │
│                     │  (ON EVENT, LISTEN) │                     │
│                     └──────────┬──────────┘                     │
│                                │                                │
│                                ▼                                │
│                     ┌─────────────────────┐                     │
│                     │  Handler Execution  │                     │
│                     │  (with filtering)   │                     │
│                     └─────────────────────┘                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Execution Context

**Important Note:** Since CheeseCake is AI-interpreted within a session:

1. **External events** (file changes, webhooks) are **declared** but require runtime integration for actual triggering
2. **Internal events** (EMIT/LISTEN) work fully within a session
3. **Schedules** are **declarative** - actual timing requires external daemon
4. **Simulation mode** allows testing event flows without external triggers

The value of this module is:
- Define syntax for future runtime implementation
- Enable internal event-driven patterns
- Document reactive workflow semantics
- Provide foundation for CheeseCake daemon (future)

---

## 2. ON EVENT Construct

### Purpose

`ON EVENT` declares an event handler that executes when a specific event occurs.

### Syntax

```cheesecake
ON EVENT event_name(param1, param2, ...) [WHERE condition]:
  # Handler body - any valid CheeseCake statements
  # Parameters are available as variables
END ON
```

### Components

| Component | Required | Description |
|-----------|----------|-------------|
| `event_name` | Yes | Identifier for the event type |
| `(params)` | Optional | Parameters passed with the event |
| `WHERE` | Optional | Filter condition (semantic or literal) |
| Body | Yes | Statements to execute when event fires |

### Event Types

#### Built-in External Events (Declarative)

These events are declared for future runtime integration:

| Event | Parameters | Description |
|-------|------------|-------------|
| `file_changed` | `path`, `type` | File system change |
| `file_created` | `path` | New file created |
| `file_deleted` | `path` | File deleted |
| `api_webhook` | `endpoint`, `payload` | HTTP webhook received |
| `timer_tick` | `timestamp` | Timer interval |
| `user_input` | `data` | User-triggered input |
| `session_start` | `session_id` | New session started |
| `session_end` | `session_id`, `result` | Session completed |

#### Custom Events

Any event name not in the built-in list is treated as a custom event:

```cheesecake
ON EVENT my_custom_event(data, metadata):
  # Handle custom event
END ON
```

### WHERE Clause

Filter which events trigger the handler:

```cheesecake
# Literal condition
ON EVENT file_changed(path, type) WHERE type == "modified":
  LOG INFO: "File modified: {path}"
END ON

# Pattern matching
ON EVENT file_changed(path) WHERE path MATCHES "src/**/*.ts":
  RUN SESSION(linter): TASK: "Lint {path}"
END ON

# Semantic condition
ON EVENT api_webhook(endpoint, payload) WHERE **{payload} contains error data**:
  RUN SESSION(error_handler): TASK: "Process error" INPUT: {payload}
END ON

# Multiple conditions
ON EVENT new_issue(issue) WHERE issue.priority == "high" AND **{issue} is urgent**:
  EMIT urgent_alert(issue: issue)
END ON
```

### Handler Rules

1. **Declaration Order**: Handlers execute in declaration order when multiple match
2. **Isolation**: Each handler has its own scope (variables don't leak)
3. **Error Handling**: Errors in one handler don't stop others
4. **No Nesting**: Cannot declare ON EVENT inside another ON EVENT
5. **No SCHEDULE Inside**: Cannot declare SCHEDULE inside ON EVENT

### Examples

#### Example 1: File Change Handler

```cheesecake
# ============================================
# File Monitoring Workflow
# ============================================

AGENT Linter:
  MODEL: sonnet
  PROMPT: "You analyze code for style and error issues."

AGENT Fixer:
  MODEL: opus
  PROMPT: "You fix code issues while preserving functionality."

VAR linter = NEW Linter()
VAR fixer = NEW Fixer()

# Handle TypeScript file changes
ON EVENT file_changed(path, type) WHERE path MATCHES "**/*.ts":
  LOG INFO: "TypeScript file {type}: {path}"

  VAR issues = RUN SESSION(linter):
    TASK: "Analyze {path} for issues"
    CONTEXT: {check_types: true, check_style: true}

  IF **{issues} contains errors**:
    LOG WARNING: "Found issues in {path}"
    RUN SESSION(fixer):
      TASK: "Fix issues in {path}"
      INPUT: {issues}
    LOG SUCCESS: "Fixed issues in {path}"
  ELSE:
    LOG INFO: "No issues in {path}"
  END IF
END ON

# Handle any file deletion
ON EVENT file_deleted(path):
  LOG WARNING: "File deleted: {path}"
  EMIT file_audit(action: "deleted", path: path, timestamp: NOW())
END ON
```

#### Example 2: Multi-Handler Coordination

```cheesecake
# Multiple handlers for same event type

# Handler 1: Log all changes
ON EVENT file_changed(path):
  LOG INFO: "[Audit] File changed: {path} at {NOW()}"
END ON

# Handler 2: Only for source files
ON EVENT file_changed(path) WHERE path MATCHES "src/**":
  EMIT source_changed(path: path)
END ON

# Handler 3: Only for config files
ON EVENT file_changed(path) WHERE path MATCHES "*.config.*":
  LOG WARNING: "Configuration changed: {path}"
  EMIT config_changed(path: path)
END ON
```

#### Example 3: API Webhook Handler

```cheesecake
AGENT IssueProcessor:
  MODEL: sonnet
  PROMPT: "You triage and categorize incoming issues."

VAR processor = NEW IssueProcessor()

ON EVENT api_webhook(endpoint, payload) WHERE endpoint == "/issues/new":
  LOG INFO: "New issue received"

  VAR triage = RUN SESSION(processor):
    TASK: "Triage this issue"
    INPUT: {payload}

  IF **{triage} indicates critical issue**:
    EMIT critical_issue(issue: payload, triage: triage)
  ELIF **{triage} indicates enhancement request**:
    EMIT enhancement_request(issue: payload, triage: triage)
  ELSE:
    EMIT normal_issue(issue: payload, triage: triage)
  END IF
END ON
```

---

## 3. SCHEDULE Construct

### Purpose

`SCHEDULE` declares a task that should run at specific times or intervals.

### Syntax

```cheesecake
SCHEDULE schedule_name:
  INTERVAL: duration | CRON: "expression" | ONCE_AT: "timestamp"
  [START_AT: "timestamp"]
  [END_AT: "timestamp"]
  TASK: statement | TASK: ... END TASK
  [RETRY: number]
  [ON_FAILURE: action]
  [ON_SUCCESS: action]
END SCHEDULE
```

### Schedule Types

#### INTERVAL

Run repeatedly at fixed intervals:

```cheesecake
SCHEDULE every_hour:
  INTERVAL: 1h
  TASK: RUN SESSION(checker): TASK: "Check status"
END SCHEDULE
```

**Duration formats:**
- `Ns` - N seconds
- `Nm` - N minutes
- `Nh` - N hours
- `Nd` - N days
- `Nw` - N weeks

#### CRON

Run on complex schedules using cron expressions:

```cheesecake
SCHEDULE weekday_morning:
  CRON: "0 9 * * 1-5"  # 9 AM, Monday-Friday
  TASK: RUN SESSION(reporter): TASK: "Generate daily report"
END SCHEDULE
```

**Cron format:** `minute hour day_of_month month day_of_week`

Common patterns:
- `0 * * * *` - Every hour at minute 0
- `0 9 * * *` - Every day at 9 AM
- `0 0 * * 0` - Every Sunday at midnight
- `*/15 * * * *` - Every 15 minutes
- `0 9 * * 1-5` - Weekdays at 9 AM

#### ONCE_AT

Run once at a specific time:

```cheesecake
SCHEDULE deployment:
  ONCE_AT: "2026-01-15T14:00:00Z"
  TASK: RUN SESSION(deployer): TASK: "Deploy to production"
END SCHEDULE
```

### Schedule Properties

| Property | Required | Default | Description |
|----------|----------|---------|-------------|
| Timing | Yes | - | INTERVAL, CRON, or ONCE_AT |
| TASK | Yes | - | Statement(s) to execute |
| START_AT | No | Now | When schedule becomes active |
| END_AT | No | Never | When schedule deactivates |
| RETRY | No | 0 | Retry count on failure |
| ON_FAILURE | No | - | Action when task fails |
| ON_SUCCESS | No | - | Action when task succeeds |

### Task Block Syntax

For simple tasks:
```cheesecake
SCHEDULE simple:
  INTERVAL: 1h
  TASK: RUN SESSION(agent): TASK: "Do something"
END SCHEDULE
```

For complex tasks:
```cheesecake
SCHEDULE complex:
  INTERVAL: 1h
  TASK:
    VAR result = RUN SESSION(agent1): TASK: "First"
    VAR processed = RUN SESSION(agent2): TASK: "Second" INPUT: {result}
    SAVE processed TO "output/result.json"
  END TASK
END SCHEDULE
```

### Examples

#### Example 1: Health Check Schedule

```cheesecake
AGENT HealthChecker:
  MODEL: sonnet
  PROMPT: "You monitor system health and report issues."

VAR checker = NEW HealthChecker()

SCHEDULE health_check:
  INTERVAL: 30m
  START_AT: "2026-01-09T00:00:00Z"
  TASK:
    VAR status = RUN SESSION(checker):
      TASK: "Check all services"
      CONTEXT: {services: ["api", "database", "cache"]}

    IF **{status} shows any service down**:
      EMIT alert(severity: "critical", status: status)
      LOG ERROR: "Service down detected"
    ELIF **{status} shows degraded performance**:
      EMIT alert(severity: "warning", status: status)
      LOG WARNING: "Performance degradation detected"
    ELSE:
      LOG INFO: "All systems healthy"
    END IF
  END TASK
  RETRY: 2
  ON_FAILURE: EMIT alert(severity: "critical", message: "Health check failed")
END SCHEDULE
```

#### Example 2: Daily Report Schedule

```cheesecake
AGENT Reporter:
  MODEL: opus
  PROMPT: "You create comprehensive daily reports."

VAR reporter = NEW Reporter()

SCHEDULE daily_report:
  CRON: "0 9 * * 1-5"  # 9 AM weekdays
  TASK:
    VAR report = RUN SESSION(reporter):
      TASK: "Generate daily status report"
      CONTEXT: {
        include_metrics: true,
        include_issues: true,
        include_progress: true
      }

    SAVE report TO "reports/daily-{TODAY()}.md"
    LOG SUCCESS: "Daily report generated"
  END TASK
  ON_SUCCESS: EMIT report_ready(date: TODAY(), path: "reports/daily-{TODAY()}.md")
END SCHEDULE
```

#### Example 3: Backup Schedule with Retry

```cheesecake
AGENT BackupManager:
  MODEL: sonnet
  PROMPT: "You manage data backups safely and efficiently."

VAR backup_mgr = NEW BackupManager()

SCHEDULE weekly_backup:
  INTERVAL: 7d
  START_AT: "2026-01-10T02:00:00Z"  # Start on Friday 2 AM
  TASK:
    LOG INFO: "Starting weekly backup..."

    VAR backup = RUN SESSION(backup_mgr):
      TASK: "Create full backup"
      CONTEXT: {
        include_database: true,
        include_files: true,
        compression: "gzip"
      }

    IF **{backup} completed successfully**:
      LOG SUCCESS: "Backup complete: {backup.size}"
    ELSE:
      THROW "Backup failed: {backup.error}"
    END IF
  END TASK
  RETRY: 3
  ON_FAILURE:
    LOG ERROR: "Backup failed after 3 attempts"
    EMIT backup_failed(timestamp: NOW())
  ON_SUCCESS:
    LOG SUCCESS: "Weekly backup completed"
    EMIT backup_complete(timestamp: NOW())
END SCHEDULE
```

#### Example 4: One-Time Scheduled Task

```cheesecake
SCHEDULE deployment_v2:
  ONCE_AT: "2026-01-20T14:00:00Z"
  TASK:
    LOG INFO: "Starting scheduled deployment..."

    # Pre-deployment checks
    VAR checks = RUN SESSION(checker): TASK: "Run pre-deployment checks"
    IF NOT **{checks} all passed**:
      THROW "Pre-deployment checks failed"
    END IF

    # Deploy
    VAR deploy = RUN SESSION(deployer): TASK: "Deploy v2.0"

    # Post-deployment validation
    VAR validation = RUN SESSION(validator): TASK: "Validate deployment"

    LOG SUCCESS: "Deployment complete"
  END TASK
  ON_FAILURE:
    LOG ERROR: "Scheduled deployment failed"
    RUN SESSION(rollback): TASK: "Rollback to previous version"
END SCHEDULE
```

---

## 4. EMIT Construct

### Purpose

`EMIT` triggers a custom event that can be caught by `ON EVENT` or `LISTEN FOR` handlers.

### Syntax

```cheesecake
EMIT event_name(param1: value1, param2: value2, ...)
```

### Rules

1. **Event Name**: Must be valid identifier
2. **Parameters**: Named parameters with values
3. **Timing**: Event is dispatched immediately
4. **Unknown Events**: Warning logged, execution continues
5. **Return**: EMIT does not return a value

### Examples

```cheesecake
# Simple event
EMIT task_started(task_id: "123")

# Event with multiple parameters
EMIT processing_complete(
  task_id: "123",
  status: "success",
  duration: elapsed_time,
  output_path: "results/output.json"
)

# Event with complex data
VAR analysis = RUN SESSION(analyzer): TASK: "Analyze data"
EMIT analysis_ready(
  data: analysis,
  timestamp: NOW(),
  source: "main-workflow"
)

# Conditional event emission
IF **{result} indicates error**:
  EMIT error_detected(
    error: result.error,
    severity: "high",
    context: current_context
  )
END IF
```

### Event Propagation

Events propagate through the handler registry:

```
EMIT task_complete(id: "123")
        │
        ▼
┌───────────────────────────────────────┐
│ Event Registry                        │
├───────────────────────────────────────┤
│ ON EVENT task_complete WHERE id < 200 │  ← Matches: Execute
│ ON EVENT task_complete WHERE id > 500 │  ← No match: Skip
│ LISTEN FOR task_complete              │  ← Matches: Execute
└───────────────────────────────────────┘
```

---

## 5. LISTEN FOR Construct

### Purpose

`LISTEN FOR` is a lightweight event listener for internal events only (those emitted via EMIT).

### Syntax

```cheesecake
LISTEN FOR event_name:
  # Handler body
  # Access event data via {event.param}
END LISTEN
```

### Differences from ON EVENT

| Feature | ON EVENT | LISTEN FOR |
|---------|----------|------------|
| External events | Yes | No |
| Internal events | Yes | Yes |
| WHERE clause | Yes | No |
| Parameters | Named | Via `event.*` |
| Use case | Full handlers | Simple listeners |

### Examples

```cheesecake
# Simple listener
LISTEN FOR data_ready:
  LOG INFO: "Data received from {event.source}"
  VAR processed = RUN SESSION(processor):
    TASK: "Process data"
    INPUT: {event.data}
END LISTEN

# Multiple listeners
LISTEN FOR task_complete:
  LOG INFO: "Task {event.task_id} complete"
END LISTEN

LISTEN FOR task_complete:
  # Different listener, same event
  EMIT audit_log(action: "task_complete", task_id: event.task_id)
END LISTEN
```

### Event Object

In LISTEN FOR handlers, event data is accessed via `event.*`:

```cheesecake
EMIT my_event(name: "test", value: 42, items: [1, 2, 3])

LISTEN FOR my_event:
  PRINT event.name    # "test"
  PRINT event.value   # 42
  PRINT event.items   # [1, 2, 3]
END LISTEN
```

---

## 6. Event Filtering

### MATCHES Operator

Pattern matching for string parameters:

```cheesecake
# Glob patterns
WHERE path MATCHES "src/**/*.ts"     # Any .ts in src/
WHERE path MATCHES "*.config.json"   # Any .config.json
WHERE path MATCHES "test_*.py"       # Files starting with test_

# Multiple patterns
WHERE path MATCHES "*.ts" OR path MATCHES "*.tsx"
```

### Comparison Operators

```cheesecake
WHERE count > 10
WHERE status == "active"
WHERE priority != "low"
WHERE timestamp >= "2026-01-01"
```

### Semantic Conditions

AI-evaluated conditions:

```cheesecake
WHERE **{payload} contains error information**
WHERE **{issue} is high priority or urgent**
WHERE **{message} appears to be spam**
```

### Combined Conditions

```cheesecake
ON EVENT file_changed(path, type)
  WHERE path MATCHES "src/**"
    AND type == "modified"
    AND **{path} is not a test file**:
  # Handler body
END ON
```

---

## 7. Event Queue Semantics

### Dispatch Order

Events are dispatched in the order they occur:

```
Time    Event
─────   ─────────────────
T1      EMIT event_a()      → Handlers for event_a execute
T2      EMIT event_b()      → Handlers for event_b execute
T3      EMIT event_a()      → Handlers for event_a execute again
```

### Handler Execution Order

Multiple handlers for the same event execute in declaration order:

```cheesecake
# Handler 1 - declared first
ON EVENT my_event():
  LOG INFO: "Handler 1"  # Executes first
END ON

# Handler 2 - declared second
ON EVENT my_event():
  LOG INFO: "Handler 2"  # Executes second
END ON

EMIT my_event()  # Output: "Handler 1", then "Handler 2"
```

### Error Isolation

Errors in handlers don't stop other handlers:

```cheesecake
ON EVENT my_event():
  THROW "Error in handler 1"  # Logged, continues to handler 2
END ON

ON EVENT my_event():
  LOG INFO: "Handler 2"  # Still executes
END ON

EMIT my_event()
# Output:
# ERROR: Error in handler 1 (in event handler)
# INFO: Handler 2
```

### No Infinite Loops

The VM tracks event chains to prevent infinite loops:

```cheesecake
ON EVENT event_a():
  EMIT event_b()
END ON

ON EVENT event_b():
  EMIT event_a()  # Would create infinite loop
END ON

# VM detects cycle and stops after MAX_EVENT_DEPTH (default: 10)
```

---

## 8. Schedule Management

### Schedule States

```
┌─────────────┐    START_AT    ┌─────────────┐    END_AT     ┌─────────────┐
│   PENDING   │ ─────────────→ │   ACTIVE    │ ────────────→ │  COMPLETED  │
└─────────────┘                └─────────────┘                └─────────────┘
                                     │
                                     │ Manual disable
                                     ▼
                               ┌─────────────┐
                               │  DISABLED   │
                               └─────────────┘
```

### Manual Triggers

Since actual scheduling requires runtime:

```bash
# Trigger a schedule manually (for testing)
/cheesecake trigger health_check

# Trigger with simulated time
/cheesecake trigger daily_report --time "2026-01-09T09:00:00Z"
```

### Schedule Listing

```bash
/cheesecake schedules

Output:
SCHEDULE            TYPE      NEXT RUN              STATUS
─────────────────────────────────────────────────────────────
health_check        INTERVAL  2026-01-09T10:30:00Z  ACTIVE
daily_report        CRON      2026-01-10T09:00:00Z  ACTIVE
deployment_v2       ONCE_AT   2026-01-20T14:00:00Z  PENDING
weekly_backup       INTERVAL  2026-01-17T02:00:00Z  ACTIVE
```

---

## 9. Integration with Other Features

### With PHASE Construct

Events can be emitted from phases:

```cheesecake
PHASE "Research":
  VAR findings = RUN SESSION(researcher): TASK: "Research"
  EMIT phase_complete(phase: "Research", output: findings)
END PHASE

ON EVENT phase_complete(phase, output):
  LOG INFO: "Phase {phase} completed"
  CHECKPOINT "{phase}-complete":
    SAVE: {output}
  END CHECKPOINT
END ON
```

### With INTERACTIVE Construct

Events during interactive mode:

```cheesecake
INTERACTIVE AT "review":
  SHOW: {draft}
  ASK USER: "Approve?"
  OPTIONS:
    - "approve" → EMIT user_approved(item: draft)
    - "reject" → EMIT user_rejected(item: draft, reason: "Manual rejection")
END INTERACTIVE

ON EVENT user_rejected(item, reason):
  LOG WARNING: "Item rejected: {reason}"
  VAR revised = RUN SESSION(editor): TASK: "Revise based on feedback"
END ON
```

### With CONFIG Block

Cost tracking for event handlers:

```cheesecake
CONFIG:
  BUDGET: $5.00
  CONFIRM_COST_ABOVE: $0.50
END CONFIG

ON EVENT large_file_changed(path):
  # This handler might trigger cost warning if expensive
  VAR analysis = RUN SESSION(analyzer):  # Opus session
    MODEL: opus
    TASK: "Deep analysis of {path}"
END ON
```

### With Checkpoints

Save state after event handling:

```cheesecake
ON EVENT batch_complete(batch_id, results):
  LOG INFO: "Batch {batch_id} complete"

  CHECKPOINT "batch-{batch_id}":
    SAVE: {results, batch_id, timestamp: NOW()}
  END CHECKPOINT
END ON
```

### With PARALLEL

Events in parallel blocks:

```cheesecake
PARALLEL:
  VAR r1 = RUN SESSION(agent1): TASK: "Task 1"
  VAR r2 = RUN SESSION(agent2): TASK: "Task 2"
END PARALLEL

# Events emitted after parallel completes
EMIT parallel_complete(results: [r1, r2])
```

---

## 10. Best Practices

### 1. Name Events Clearly

```cheesecake
# Good - descriptive names
EMIT user_registration_complete(user_id: id)
EMIT payment_processing_failed(order_id: id, error: err)

# Avoid - vague names
EMIT done(x: data)
EMIT error(e: err)
```

### 2. Include Relevant Context

```cheesecake
# Good - includes context
EMIT task_failed(
  task_id: id,
  error: error,
  timestamp: NOW(),
  retry_count: attempts,
  context: current_context
)

# Avoid - missing context
EMIT task_failed(error: error)
```

### 3. Use WHERE Clauses Efficiently

```cheesecake
# Good - specific filter
ON EVENT file_changed(path) WHERE path MATCHES "src/**/*.ts":
  # Only handles TypeScript source files
END ON

# Avoid - filter in handler body
ON EVENT file_changed(path):
  IF path MATCHES "src/**/*.ts":
    # Wasteful - handler fires for all file changes
  END IF
END ON
```

### 4. Handle Errors in Handlers

```cheesecake
ON EVENT critical_task(data):
  TRY:
    VAR result = RUN SESSION(processor): TASK: "Process" INPUT: {data}
  CATCH error:
    LOG ERROR: "Handler failed: {error}"
    EMIT handler_error(handler: "critical_task", error: error)
  END TRY
END ON
```

### 5. Avoid Event Loops

```cheesecake
# Dangerous - potential infinite loop
ON EVENT event_a():
  EMIT event_b()
END ON

ON EVENT event_b():
  EMIT event_a()  # Creates loop!
END ON

# Safe - use condition to break cycle
ON EVENT event_a(depth):
  IF depth < 5:
    EMIT event_b(depth: depth + 1)
  END IF
END ON
```

### 6. Document Event Contracts

```cheesecake
# ============================================
# Event: user_action
# Parameters:
#   - user_id: string - The user identifier
#   - action: string - Action type (create, update, delete)
#   - data: object - Action-specific data
# Emitted by: user interaction handlers
# Handled by: audit, analytics, notification systems
# ============================================
ON EVENT user_action(user_id, action, data):
  # Handler implementation
END ON
```

### 7. Use Schedules for Recurring Tasks

```cheesecake
# Good - schedule for recurring
SCHEDULE hourly_cleanup:
  INTERVAL: 1h
  TASK: RUN SESSION(cleaner): TASK: "Clean temp files"
END SCHEDULE

# Avoid - manual loop
LOOP:
  RUN SESSION(cleaner): TASK: "Clean temp files"
  WAIT 3600s  # Not a real construct
END LOOP
```

### 8. Test Event Handlers

```cheesecake
# Test by emitting test events
TEST "file change handler works":
  # Emit test event
  EMIT file_changed(path: "test/sample.ts", type: "modified")

  # Verify handler executed (check side effects)
  ASSERT log_contains("File modified: test/sample.ts")
END TEST
```

---

## 11. Examples

### Example 1: Complete Event-Driven Workflow

```cheesecake
# ============================================
# Event-Driven Data Pipeline
# ============================================

# Agents
AGENT DataFetcher:
  MODEL: sonnet
  PROMPT: "You fetch and validate data from sources."

AGENT DataProcessor:
  MODEL: sonnet
  PROMPT: "You transform and process data."

AGENT DataValidator:
  MODEL: sonnet
  PROMPT: "You validate data quality."

AGENT AlertManager:
  MODEL: sonnet
  PROMPT: "You manage alerts and notifications."

# Agent instances
VAR fetcher = NEW DataFetcher()
VAR processor = NEW DataProcessor()
VAR validator = NEW DataValidator()
VAR alerter = NEW AlertManager()

# ============================================
# Event Handlers
# ============================================

# Handle new data source events
ON EVENT data_source_available(source, url):
  LOG INFO: "New data source: {source}"

  VAR data = RUN SESSION(fetcher):
    TASK: "Fetch data from {url}"
    CONTEXT: {source: source}

  IF **{data} fetched successfully**:
    EMIT data_fetched(source: source, data: data)
  ELSE:
    EMIT fetch_failed(source: source, error: data.error)
  END IF
END ON

# Handle fetched data
ON EVENT data_fetched(source, data):
  LOG INFO: "Processing data from {source}"

  VAR processed = RUN SESSION(processor):
    TASK: "Process and transform data"
    INPUT: {data}

  EMIT data_processed(source: source, data: processed)
END ON

# Handle processed data
ON EVENT data_processed(source, data):
  LOG INFO: "Validating data from {source}"

  VAR validation = RUN SESSION(validator):
    TASK: "Validate data quality"
    INPUT: {data}

  IF **{validation} passed all checks**:
    EMIT data_ready(source: source, data: data, validation: validation)
  ELSE:
    EMIT validation_failed(source: source, errors: validation.errors)
  END IF
END ON

# Handle ready data
ON EVENT data_ready(source, data, validation):
  LOG SUCCESS: "Data pipeline complete for {source}"
  SAVE data TO "output/{source}-{TODAY()}.json"
  EMIT pipeline_complete(source: source, timestamp: NOW())
END ON

# Handle failures
ON EVENT fetch_failed(source, error):
  LOG ERROR: "Fetch failed for {source}: {error}"
  RUN SESSION(alerter):
    TASK: "Send alert about fetch failure"
    INPUT: {source: source, error: error}
END ON

ON EVENT validation_failed(source, errors):
  LOG ERROR: "Validation failed for {source}"
  RUN SESSION(alerter):
    TASK: "Send alert about validation failure"
    INPUT: {source: source, errors: errors}
END ON

# ============================================
# Trigger the pipeline
# ============================================

EMIT data_source_available(source: "weather-api", url: "https://api.weather.com/data")
EMIT data_source_available(source: "stock-data", url: "https://api.stocks.com/daily")
```

### Example 2: Scheduled Monitoring System

```cheesecake
# ============================================
# Scheduled Monitoring System
# ============================================

CONFIG:
  BUDGET: $10.00
  WARN_AT_PERCENT: 80
END CONFIG

AGENT SystemMonitor:
  MODEL: sonnet
  PROMPT: "You monitor system health and performance."

AGENT ReportGenerator:
  MODEL: opus
  PROMPT: "You create detailed monitoring reports."

AGENT IncidentResponder:
  MODEL: sonnet
  PROMPT: "You respond to and document incidents."

VAR monitor = NEW SystemMonitor()
VAR reporter = NEW ReportGenerator()
VAR responder = NEW IncidentResponder()

# ============================================
# Schedules
# ============================================

# Health check every 15 minutes
SCHEDULE health_check:
  INTERVAL: 15m
  TASK:
    VAR status = RUN SESSION(monitor):
      TASK: "Check system health"
      CONTEXT: {
        check_cpu: true,
        check_memory: true,
        check_disk: true,
        check_network: true
      }

    IF **{status} shows critical issues**:
      EMIT critical_alert(status: status)
    ELIF **{status} shows warnings**:
      EMIT warning_alert(status: status)
    END IF
  END TASK
  RETRY: 2
  ON_FAILURE: EMIT monitor_failure(schedule: "health_check")
END SCHEDULE

# Daily summary report
SCHEDULE daily_summary:
  CRON: "0 18 * * *"  # 6 PM daily
  TASK:
    VAR report = RUN SESSION(reporter):
      TASK: "Generate daily monitoring summary"
      CONTEXT: {
        include_metrics: true,
        include_incidents: true,
        include_trends: true
      }

    SAVE report TO "reports/daily-{TODAY()}.md"
    EMIT daily_report_ready(path: "reports/daily-{TODAY()}.md")
  END TASK
END SCHEDULE

# Weekly detailed analysis
SCHEDULE weekly_analysis:
  CRON: "0 9 * * 1"  # Monday 9 AM
  TASK:
    VAR analysis = RUN SESSION(reporter):
      TASK: "Generate weekly analysis"
      CONTEXT: {
        compare_to_previous: true,
        identify_trends: true,
        suggest_improvements: true
      }

    SAVE analysis TO "reports/weekly-{WEEK()}.md"
  END TASK
END SCHEDULE

# ============================================
# Alert Handlers
# ============================================

ON EVENT critical_alert(status):
  LOG ERROR: "CRITICAL: System issues detected"

  VAR response = RUN SESSION(responder):
    TASK: "Document and respond to critical incident"
    INPUT: {status}
    CONTEXT: {priority: "P1"}

  CHECKPOINT "incident-{NOW()}":
    SAVE: {status, response}
  END CHECKPOINT
END ON

ON EVENT warning_alert(status):
  LOG WARNING: "Warning: Performance degradation detected"

  RUN SESSION(responder):
    TASK: "Log warning for review"
    INPUT: {status}
END ON

ON EVENT monitor_failure(schedule):
  LOG ERROR: "Monitor schedule {schedule} failed"
  # Could trigger backup monitoring
END ON
```

### Example 3: Internal Event Coordination

```cheesecake
# ============================================
# Multi-Stage Processing with Events
# ============================================

AGENT Stage1:
  MODEL: sonnet
  PROMPT: "You handle stage 1 processing."

AGENT Stage2:
  MODEL: sonnet
  PROMPT: "You handle stage 2 processing."

AGENT Stage3:
  MODEL: opus
  PROMPT: "You handle final stage processing."

VAR s1 = NEW Stage1()
VAR s2 = NEW Stage2()
VAR s3 = NEW Stage3()

# Track progress
VAR stages_complete = []

# Stage listeners
LISTEN FOR stage1_complete:
  LOG INFO: "Stage 1 complete, starting Stage 2"
  stages_complete = APPEND(stages_complete, "stage1")

  VAR result = RUN SESSION(s2):
    TASK: "Process stage 2"
    INPUT: {event.data}

  EMIT stage2_complete(data: result)
END LISTEN

LISTEN FOR stage2_complete:
  LOG INFO: "Stage 2 complete, starting Stage 3"
  stages_complete = APPEND(stages_complete, "stage2")

  VAR result = RUN SESSION(s3):
    TASK: "Final processing"
    INPUT: {event.data}

  EMIT stage3_complete(data: result)
END LISTEN

LISTEN FOR stage3_complete:
  LOG SUCCESS: "All stages complete!"
  stages_complete = APPEND(stages_complete, "stage3")
  SAVE event.data TO "output/final-result.json"
END LISTEN

# Start the pipeline
VAR initial = RUN SESSION(s1):
  TASK: "Initial processing"
  INPUT: {source: "input.json"}

EMIT stage1_complete(data: initial)

LOG INFO: "Stages completed: {stages_complete}"
```

---

## 12. VM Implementation Guide

### Event Registration

During parsing, the VM should:

1. Collect all ON EVENT declarations
2. Collect all LISTEN FOR declarations
3. Build handler registry:

```
EventRegistry = {
  "file_changed": [
    {handler: handler1, where: "path MATCHES 'src/**'"},
    {handler: handler2, where: null}
  ],
  "task_complete": [
    {handler: handler3, where: "status == 'success'"}
  ]
}
```

### Event Dispatch Protocol

When EMIT is called:

```
1. Look up event_name in EventRegistry
2. For each handler:
   a. Evaluate WHERE clause (if present)
   b. If matches:
      - Create handler scope
      - Bind event parameters
      - Execute handler body
      - Catch and log any errors
      - Continue to next handler
3. Return (EMIT has no return value)
```

### Schedule Registration

During parsing:

```
ScheduleRegistry = {
  "health_check": {
    type: "INTERVAL",
    value: "15m",
    task: taskBlock,
    retry: 2,
    onFailure: failureAction
  },
  "daily_report": {
    type: "CRON",
    value: "0 9 * * *",
    task: taskBlock
  }
}
```

### Manual Trigger Protocol

For `/cheesecake trigger <name>`:

1. Look up schedule in ScheduleRegistry
2. Execute task block
3. Handle retry logic if task fails
4. Execute ON_SUCCESS or ON_FAILURE

---

## 13. Limitations & Future Work

### Current Limitations

1. **No Real File Watching**: File events are declarative only
2. **No Background Scheduling**: Schedules require manual trigger or daemon
3. **No Persistent Event Queue**: Events only exist within session
4. **No Cross-Session Events**: Events don't persist across sessions

### Future Enhancements (CheeseCake Daemon)

A future CheeseCake daemon could provide:

1. **File System Monitoring**: Real file_changed events
2. **Background Scheduling**: True cron-like execution
3. **Webhook Server**: Receive external HTTP events
4. **Persistent Queue**: Events survive session restarts
5. **Cross-Workflow Events**: Events between different workflows

### Workarounds

For now, use these patterns:

```cheesecake
# Poll for file changes
SCHEDULE file_check:
  INTERVAL: 1m
  TASK:
    VAR files = LIST_CHANGED_FILES(since: LAST_CHECK())
    FOR file IN files:
      EMIT file_changed(path: file.path, type: file.type)
    END FOR
  END TASK
END SCHEDULE

# Manual event simulation
# Run: /cheesecake emit file_changed path="src/app.ts" type="modified"
```

---

## 14. Quick Reference

### ON EVENT

```cheesecake
ON EVENT name(params) [WHERE condition]:
  # Handler body
END ON
```

### SCHEDULE

```cheesecake
SCHEDULE name:
  INTERVAL: Nh | CRON: "expr" | ONCE_AT: "timestamp"
  [START_AT: "timestamp"]
  [END_AT: "timestamp"]
  TASK: statement | TASK: ... END TASK
  [RETRY: N]
  [ON_FAILURE: action]
  [ON_SUCCESS: action]
END SCHEDULE
```

### EMIT

```cheesecake
EMIT event_name(param: value, ...)
```

### LISTEN FOR

```cheesecake
LISTEN FOR event_name:
  # Use event.param to access data
END LISTEN
```

---

## 15. Glossary

| Term | Definition |
|------|------------|
| **Event** | A named occurrence with associated data |
| **Handler** | Code that executes in response to an event |
| **Schedule** | A time-based trigger configuration |
| **EMIT** | Trigger a custom event |
| **LISTEN** | Register a lightweight event handler |
| **WHERE Clause** | Filter condition for event handlers |
| **Event Registry** | Internal mapping of events to handlers |
| **Event Queue** | Ordered list of pending events |

---

**Module 12: Events & Scheduling - Complete Specification**
