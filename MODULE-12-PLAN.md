# Module 12: Events & Scheduling - Implementation Plan

**Module**: Events & Scheduling (v0.0.2)
**Status**: In Progress
**Date**: 2026-01-09

---

## Overview

Module 12 adds reactive programming patterns to CheeseCake through event handlers and scheduled tasks. This enables automation workflows that respond to external triggers and run on schedules.

---

## Features

### 1. ON EVENT Construct

Event handlers that respond to triggers:

```cheesecake
ON EVENT file_changed(path) WHERE path MATCHES "src/**/*.ts":
  VAR lint_result = RUN SESSION(linter): TASK: "Check {path}"
  IF **{lint_result} has errors**:
    RUN SESSION(fixer): TASK: "Fix lint errors in {path}"
  END IF
END ON

ON EVENT new_issue(issue):
  RUN SESSION(triage): TASK: "Triage and assign {issue}"
END ON
```

**Event Types:**
- `file_changed(path)` - File system changes
- `new_issue(issue)` - Issue tracker events
- `api_response(response)` - API webhook events
- `user_input(data)` - User-triggered events
- `timer_tick(timestamp)` - Timer events
- Custom events via EMIT

**Event Properties:**
- `WHERE` clause for filtering
- Event data accessible as parameters
- Multiple handlers per event type
- Priority ordering (optional)

### 2. SCHEDULE Construct

Time-based task scheduling:

```cheesecake
SCHEDULE health_check:
  INTERVAL: 1h
  TASK: RUN SESSION(monitor): TASK: "Check system health"
  ON_FAILURE: NOTIFY "ops@company.com"
END SCHEDULE

SCHEDULE daily_report:
  CRON: "0 9 * * *"  # 9 AM daily
  TASK: RUN SESSION(reporter): TASK: "Generate daily report"
END SCHEDULE

SCHEDULE weekly_backup:
  INTERVAL: 7d
  START_AT: "2026-01-10T00:00:00Z"
  TASK: RUN SESSION(backup): TASK: "Create weekly backup"
  RETRY: 3
END SCHEDULE
```

**Schedule Types:**
- `INTERVAL` - Fixed time between runs (1m, 1h, 1d, etc.)
- `CRON` - Cron expression for complex schedules
- `ONCE_AT` - Single execution at specific time

**Schedule Properties:**
- `START_AT` - When to start the schedule
- `END_AT` - When to stop the schedule
- `RETRY` - Number of retries on failure
- `ON_FAILURE` - Action on failure
- `ON_SUCCESS` - Action on success

### 3. EMIT Construct

Emit custom events:

```cheesecake
# Emit event for other handlers to catch
EMIT task_completed(data: {task_id: "123", status: "success"})

# Emit event with timestamp
EMIT log_event(level: "INFO", message: "Processing started", timestamp: NOW())
```

### 4. Event Bus

Internal event coordination:

```cheesecake
# Register event listener
LISTEN FOR data_ready:
  VAR processed = RUN SESSION(processor): TASK: "Process {event.data}"
END LISTEN

# Emit to trigger listeners
EMIT data_ready(data: {items: [1, 2, 3]})
```

---

## Implementation Scope

### In Scope for v0.0.2
- ON EVENT syntax and semantics
- SCHEDULE syntax (INTERVAL only)
- EMIT for custom events
- LISTEN FOR internal events
- Basic event filtering (WHERE clause)
- Documentation and examples

### Out of Scope (Future)
- Actual file system monitoring (requires runtime integration)
- Real CRON scheduling (requires background process)
- External webhook integration
- Persistent event queues
- Event replay/history

**Note:** Since CheeseCake is AI-interpreted, the VM can only simulate event handling within a session. True background scheduling requires runtime integration outside CheeseCake.

---

## Files to Create/Modify

### 1. NEW: skills/cheesecake/events.md (Target: 800+ lines)
Complete specification for events and scheduling:
- ON EVENT syntax and semantics
- SCHEDULE syntax and semantics
- EMIT and LISTEN constructs
- Event types and filtering
- Execution protocols
- Examples and best practices

### 2. UPDATE: skills/cheesecake/SKILL.md (+200 lines)
Add new section 12: "Events & Scheduling (v0.0.2+)"
- ON EVENT construct syntax
- SCHEDULE construct syntax
- EMIT construct syntax
- LISTEN FOR construct syntax
- Rules and constraints
- Examples

Renumber existing sections:
- 12 → 13 (Error Handling)
- 13 → 14 (Functions)
- 14 → 15 (Modules & Imports)
- 15 → 16 (Built-in Functions)
- 16 → 17 (Operators)
- 17 → 18 (Best Practices)
- 18 → 19 (Complete Example)

### 3. UPDATE: skills/cheesecake/vm.md (+300 lines)
Add new section 10: "Event & Schedule Execution (v0.0.2+)"
- Event registration protocol
- Event dispatch protocol
- Schedule management
- Event queue semantics
- Execution priority

Renumber existing sections:
- 10 → 11 (Variable & Scope Management)
- 11 → 12 (State Persistence)
- 12 → 13 (Semantic Condition Evaluation)
- ... and so on

### 4. NEW: test-events.cheesecake (Target: 200+ lines)
Test file covering:
- ON EVENT declaration and handling
- SCHEDULE declaration
- EMIT custom events
- LISTEN FOR internal events
- Event filtering with WHERE
- Multiple handlers
- Error handling in events

### 5. UPDATE: CHANGELOG.md (+80 lines)
Document Module 12 completion

### 6. NEW: MODULE-12-COMPLETE.md
Completion marker with summary

---

## Syntax Specification

### ON EVENT

```
ON EVENT event_name(parameters) [WHERE condition]:
  # Handler body - any valid CheeseCake statements
END ON
```

**Rules:**
- Event name must be valid identifier
- Parameters become available in handler body
- WHERE clause is optional, uses semantic conditions
- Multiple handlers for same event execute in declaration order
- Handlers cannot contain SCHEDULE blocks
- Handlers can emit other events (but beware infinite loops)

### SCHEDULE

```
SCHEDULE schedule_name:
  INTERVAL: duration | CRON: "cron_expr" | ONCE_AT: "timestamp"
  [START_AT: "timestamp"]
  [END_AT: "timestamp"]
  TASK: statement | block
  [RETRY: number]
  [ON_FAILURE: action]
  [ON_SUCCESS: action]
END SCHEDULE
```

**Rules:**
- Exactly one of INTERVAL, CRON, or ONCE_AT required
- TASK is required
- Duration format: Nm (minutes), Nh (hours), Nd (days)
- CRON uses standard 5-field format
- RETRY defaults to 0

### EMIT

```
EMIT event_name(param1: value1, param2: value2, ...)
```

**Rules:**
- Event name must match registered handler
- Parameters become event.param_name in handler
- Emitting unknown event is no-op (warning only)

### LISTEN FOR

```
LISTEN FOR event_name:
  # Handler using {event.param} syntax
END LISTEN
```

**Rules:**
- Same as ON EVENT but for internal events only
- Can only catch events emitted with EMIT
- Lighter weight than ON EVENT

---

## Execution Model

### Event Registration Phase

During parsing, the VM:
1. Collects all ON EVENT declarations
2. Collects all SCHEDULE declarations
3. Collects all LISTEN FOR declarations
4. Builds event handler registry

### Event Dispatch

When an event occurs:
1. Find matching handlers in registry
2. Filter by WHERE clause (if present)
3. Execute handlers in declaration order
4. Capture any errors per handler
5. Continue to next handler even if one fails

### Schedule Simulation

Since true background scheduling isn't possible in AI interpretation:
1. SCHEDULE blocks are validated but not auto-triggered
2. User can manually trigger: `/cheesecake trigger <schedule_name>`
3. For demonstration, simulate schedule execution flow

---

## Examples

### Example 1: File Change Handler

```cheesecake
# Handler for TypeScript file changes
ON EVENT file_changed(path) WHERE path MATCHES "**/*.ts":
  LOG INFO: "File changed: {path}"

  VAR linter = NEW Linter()
  VAR result = RUN SESSION(linter): TASK: "Lint {path}"

  IF **{result} has errors**:
    EMIT lint_errors(path: path, errors: result.errors)
  END IF
END ON

# Handler for lint errors
ON EVENT lint_errors(path, errors):
  LOG WARNING: "{errors.length} errors in {path}"

  VAR fixer = NEW AutoFixer()
  RUN SESSION(fixer): TASK: "Fix errors in {path}" INPUT: {errors}
END ON
```

### Example 2: Scheduled Task

```cheesecake
AGENT HealthChecker:
  MODEL: sonnet
  PROMPT: "Check system health and report status"

VAR checker = NEW HealthChecker()

SCHEDULE hourly_health_check:
  INTERVAL: 1h
  START_AT: "2026-01-09T00:00:00Z"
  TASK:
    VAR status = RUN SESSION(checker): TASK: "Check all services"
    IF **{status} indicates problems**:
      EMIT alert(severity: "high", message: status)
    END IF
  END TASK
  RETRY: 2
  ON_FAILURE: LOG ERROR: "Health check failed after retries"
END SCHEDULE
```

### Example 3: Internal Event Communication

```cheesecake
# Producer workflow
VAR data = RUN SESSION(fetcher): TASK: "Fetch data"
EMIT data_ready(data: data, source: "api")

# Consumer (in same file or imported)
LISTEN FOR data_ready:
  LOG INFO: "Received data from {event.source}"
  VAR processed = RUN SESSION(processor): TASK: "Process" INPUT: {event.data}
  SAVE processed TO "output/processed.json"
END LISTEN
```

---

## Success Criteria

- [ ] ON EVENT syntax defined and documented
- [ ] SCHEDULE syntax defined and documented
- [ ] EMIT and LISTEN FOR defined
- [ ] Event filtering with WHERE works
- [ ] VM execution semantics documented
- [ ] Test file covers all features
- [ ] CHANGELOG updated
- [ ] All existing tests still pass
- [ ] Backward compatible (events are optional)

---

## Estimated Line Counts

| File | Lines |
|------|-------|
| events.md | 800+ |
| SKILL.md additions | 200+ |
| vm.md additions | 300+ |
| test-events.cheesecake | 200+ |
| CHANGELOG additions | 80+ |
| MODULE-12-COMPLETE.md | 300+ |
| **Total** | **1,880+** |

---

## Implementation Order

1. Create events.md (complete specification)
2. Update SKILL.md (add section 12, renumber)
3. Update vm.md (add section 10, renumber)
4. Create test-events.cheesecake
5. Update CHANGELOG.md
6. Create MODULE-12-COMPLETE.md
7. Commit and push

---

## Notes

Since CheeseCake is AI-interpreted and runs within a single session:

1. **File system events** can only be simulated - no actual file watching
2. **Schedules** are declarative - actual scheduling requires external runtime
3. **Events** are primarily for workflow coordination within a session
4. **Future**: A CheeseCake daemon could provide real event monitoring

The primary value of Module 12 is:
- Documenting the event/schedule syntax for future runtime implementation
- Enabling internal event-driven patterns within workflows
- Providing a foundation for reactive workflows

---

Ready to begin implementation!
