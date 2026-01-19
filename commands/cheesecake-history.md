# /cheesecake history
# Purpose: View and manage execution history
# Part of: CheeseCake v0.0.2 - Module 14 (History & Replay)
#
# This command displays execution history, shows details of specific
# executions, and manages history cleanup.
#
# Usage:
#   /cheesecake history              - List recent executions
#   /cheesecake history #N           - Show details for execution #N
#   /cheesecake history --clear      - Clear history
#
# Dependencies:
#   - skills/cheesecake/history.md
#   - skills/cheesecake/vm.md
#
# Related:
#   - commands/cheesecake-replay.md

---
description: View and manage CheeseCake execution history
---

# History Command

Display and manage the execution history of CheeseCake programs.

---

## Basic Usage

### List Recent Executions

```
/cheesecake history
```

**Output:**
```
┌─────────────────────────────────────────────────────────────────────────────┐
│  CheeseCake Execution History                                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  #   Date/Time          Program                    Status   Duration  Cost  │
│  ─── ────────────────── ────────────────────────── ──────── ──────── ────── │
│  1   2026-01-14 14:20   pipeline.cheesecake        ✓ Done   2m 45s   $0.12 │
│  2   2026-01-14 11:45   research.cheesecake        ✓ Done   1m 30s   $0.08 │
│  3   2026-01-14 10:30   workflow.cheesecake        ✗ Failed 0m 45s   $0.03 │
│  4   2026-01-13 16:00   data-processor.cheesecake  ✓ Done   5m 12s   $0.25 │
│  5   2026-01-13 14:30   report-gen.cheesecake      ✓ Done   3m 20s   $0.15 │
│                                                                             │
│  Showing 5 of 47 executions                                                 │
│                                                                             │
│  Commands:                                                                  │
│    /cheesecake history #N        View execution details                     │
│    /cheesecake history --all     Show all executions                        │
│    /cheesecake replay #N         Replay an execution                        │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### View Execution Details

```
/cheesecake history #1
```

**Output:**
```
┌─────────────────────────────────────────────────────────────────────────────┐
│  Execution #1: pipeline.cheesecake                                          │
│  ID: a1b2c3                                                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Status:      ✓ Success                                                     │
│  Started:     2026-01-14 14:20:00                                          │
│  Completed:   2026-01-14 14:22:45                                          │
│  Duration:    2m 45s                                                        │
│                                                                             │
│  ─────────────────────────────────────────────────────────────────────────  │
│  COST BREAKDOWN                                                             │
│  ─────────────────────────────────────────────────────────────────────────  │
│                                                                             │
│  Total Cost:     $0.12                                                      │
│  Sessions:       5                                                          │
│  Tokens:         6,800 (4,500 input + 2,300 output)                        │
│                                                                             │
│  By Model:                                                                  │
│    sonnet        4 sessions       $0.08                                    │
│    opus          1 session        $0.04                                    │
│                                                                             │
│  ─────────────────────────────────────────────────────────────────────────  │
│  INPUTS                                                                     │
│  ─────────────────────────────────────────────────────────────────────────  │
│                                                                             │
│    topic:        "machine learning trends"                                 │
│    depth:        "deep"                                                    │
│    max_sources:  10                                                        │
│                                                                             │
│  ─────────────────────────────────────────────────────────────────────────  │
│  OUTPUTS                                                                    │
│  ─────────────────────────────────────────────────────────────────────────  │
│                                                                             │
│    findings:     "Analysis of ML trends shows significant growth in..."    │
│    sources:      ["arxiv.org/paper1", "nature.com/article2", ...]         │
│    confidence:   0.92                                                      │
│                                                                             │
│  ─────────────────────────────────────────────────────────────────────────  │
│  PHASES                                                                     │
│  ─────────────────────────────────────────────────────────────────────────  │
│                                                                             │
│    ✓ Research     45s    2 sessions   $0.04                                │
│    ✓ Analysis     60s    2 sessions   $0.05                                │
│    ✓ Output       60s    1 session    $0.03                                │
│                                                                             │
│  ─────────────────────────────────────────────────────────────────────────  │
│  CHECKPOINTS                                                                │
│  ─────────────────────────────────────────────────────────────────────────  │
│                                                                             │
│    ✓ research-complete     Saved: raw_data, sources                        │
│    ✓ analysis-complete     Saved: findings, confidence                     │
│                                                                             │
│  ─────────────────────────────────────────────────────────────────────────  │
│  COMMANDS                                                                   │
│  ─────────────────────────────────────────────────────────────────────────  │
│                                                                             │
│    /cheesecake replay #1                    Replay with same inputs        │
│    /cheesecake replay #1 --modify           Replay with modified inputs    │
│    /cheesecake replay #1 --from research-complete   Resume from checkpoint │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### View Failed Execution

```
/cheesecake history #3
```

**Output:**
```
┌─────────────────────────────────────────────────────────────────────────────┐
│  Execution #3: workflow.cheesecake                                          │
│  ID: x1y2z3                                                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Status:      ✗ FAILED                                                      │
│  Started:     2026-01-14 10:30:00                                          │
│  Failed at:   2026-01-14 10:30:45                                          │
│  Duration:    45s (before failure)                                         │
│                                                                             │
│  ─────────────────────────────────────────────────────────────────────────  │
│  ERROR DETAILS                                                              │
│  ─────────────────────────────────────────────────────────────────────────  │
│                                                                             │
│  Type:        SessionError                                                  │
│  Message:     API rate limit exceeded                                       │
│  Phase:       Research                                                      │
│  Line:        45                                                            │
│                                                                             │
│  Statement:                                                                 │
│    VAR data = RUN SESSION(researcher): TASK: "Fetch data"                  │
│                                                                             │
│  Stack trace:                                                               │
│    at Research phase, line 45                                              │
│    at PARALLEL block, line 40                                              │
│    at main execution                                                       │
│                                                                             │
│  ─────────────────────────────────────────────────────────────────────────  │
│  PARTIAL OUTPUTS                                                            │
│  ─────────────────────────────────────────────────────────────────────────  │
│                                                                             │
│    sources:      ["partial-source-1"]  (incomplete)                        │
│                                                                             │
│  ─────────────────────────────────────────────────────────────────────────  │
│  PHASES COMPLETED                                                           │
│  ─────────────────────────────────────────────────────────────────────────  │
│                                                                             │
│    ✓ Setup       5s     1 session    $0.01                                 │
│    ✗ Research    40s    2 sessions   $0.02   ← Failed here                 │
│                                                                             │
│  ─────────────────────────────────────────────────────────────────────────  │
│  RECOVERY OPTIONS                                                           │
│  ─────────────────────────────────────────────────────────────────────────  │
│                                                                             │
│    /cheesecake replay #3               Retry from beginning                │
│    /cheesecake replay #3 --from setup-complete   Resume after Setup phase  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Filtering Options

### By Status

```
/cheesecake history --status failed
```

Show only failed executions.

```
/cheesecake history --status success
```

Show only successful executions.

### By Program

```
/cheesecake history --program workflow
```

Show executions of programs matching "workflow".

```
/cheesecake history --program research.cheesecake
```

Show executions of specific program.

### By Date

```
/cheesecake history --since 2026-01-01
```

Show executions since January 1st.

```
/cheesecake history --before 2026-01-10
```

Show executions before January 10th.

```
/cheesecake history --since 2026-01-01 --before 2026-01-15
```

Show executions in date range.

### By Cost

```
/cheesecake history --cost-above 0.50
```

Show executions that cost more than $0.50.

```
/cheesecake history --cost-below 0.10
```

Show executions that cost less than $0.10.

### By Tag

```
/cheesecake history --tag production
```

Show executions tagged with "production".

### Combined Filters

```
/cheesecake history --status failed --program workflow --since 2026-01-01
```

Show failed workflow executions since January 1st.

---

## Output Formats

### Default (Table)

```
/cheesecake history
```

Human-readable table format.

### JSON

```
/cheesecake history --json
```

**Output:**
```json
{
  "executions": [
    {
      "id": "a1b2c3",
      "program": "pipeline.cheesecake",
      "status": "success",
      "started_at": "2026-01-14T14:20:00Z",
      "duration_ms": 165000,
      "cost_usd": 0.12
    },
    ...
  ],
  "total": 47,
  "showing": 10
}
```

### Compact

```
/cheesecake history --compact
```

**Output:**
```
#1 pipeline.cheesecake ✓ 2m45s $0.12
#2 research.cheesecake ✓ 1m30s $0.08
#3 workflow.cheesecake ✗ 0m45s $0.03
```

---

## History Cleanup

### Clear All History

```
/cheesecake history --clear
```

**Output:**
```
⚠️  This will permanently delete all execution history (47 entries).

Are you sure? [y/N]: y

✓ Cleared 47 history entries.
```

### Clear with Filters

```
/cheesecake history --clear --before 2026-01-01
```

Clear entries older than January 1st.

```
/cheesecake history --clear --status failed
```

Clear only failed executions.

```
/cheesecake history --clear --program test.cheesecake
```

Clear history for specific program.

### Dry-Run Clear

```
/cheesecake history --clear --dry-run
```

**Output:**
```
DRY RUN: Would delete 47 history entries.

Breakdown:
  - 35 successful executions
  - 12 failed executions
  - Oldest: 2025-12-15
  - Newest: 2026-01-14

To actually delete, run without --dry-run.
```

---

## Statistics

### View History Stats

```
/cheesecake history --stats
```

**Output:**
```
┌─────────────────────────────────────────────────────────────────────────────┐
│  History Statistics                                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Total Executions:    47                                                    │
│  Successful:          35 (74%)                                              │
│  Failed:              12 (26%)                                              │
│                                                                             │
│  Date Range:          2025-12-15 to 2026-01-14 (30 days)                   │
│                                                                             │
│  ─────────────────────────────────────────────────────────────────────────  │
│  COST SUMMARY                                                               │
│  ─────────────────────────────────────────────────────────────────────────  │
│                                                                             │
│  Total Cost:          $4.87                                                │
│  Average per Run:     $0.10                                                │
│  Highest:             $0.85 (#12 data-pipeline.cheesecake)                 │
│  Lowest:              $0.01 (#45 hello.cheesecake)                         │
│                                                                             │
│  ─────────────────────────────────────────────────────────────────────────  │
│  DURATION SUMMARY                                                           │
│  ─────────────────────────────────────────────────────────────────────────  │
│                                                                             │
│  Total Time:          2h 34m                                               │
│  Average per Run:     3m 17s                                               │
│  Longest:             15m 42s (#12 data-pipeline.cheesecake)               │
│  Shortest:            12s (#45 hello.cheesecake)                           │
│                                                                             │
│  ─────────────────────────────────────────────────────────────────────────  │
│  TOP PROGRAMS                                                               │
│  ─────────────────────────────────────────────────────────────────────────  │
│                                                                             │
│  Program                      Runs    Success Rate    Avg Cost              │
│  ─────────────────────────── ─────── ──────────────── ─────────            │
│  workflow.cheesecake          15      80%             $0.12                │
│  research.cheesecake          12      92%             $0.08                │
│  pipeline.cheesecake          8       75%             $0.18                │
│  report-gen.cheesecake        7       100%            $0.15                │
│  data-processor.cheesecake    5       60%             $0.25                │
│                                                                             │
│  ─────────────────────────────────────────────────────────────────────────  │
│  COMMON ERRORS                                                              │
│  ─────────────────────────────────────────────────────────────────────────  │
│                                                                             │
│  Error Type               Count    Last Occurrence                          │
│  ─────────────────────── ─────── ──────────────────                        │
│  SessionError             5       2026-01-14 10:30                         │
│  TimeoutError             4       2026-01-13 09:15                         │
│  ValidationError          2       2026-01-12 14:00                         │
│  BudgetExceeded           1       2026-01-10 11:30                         │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Comparison

### Compare Two Executions

```
/cheesecake history --compare #1 #5
```

**Output:**
```
┌─────────────────────────────────────────────────────────────────────────────┐
│  Execution Comparison: #1 vs #5                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│                          #1                    #5                           │
│  ──────────────────────────────────────────────────────────────────────    │
│  Program:               pipeline.cheesecake   pipeline.cheesecake          │
│  Status:                ✓ Success             ✓ Success                    │
│  Date:                  2026-01-14 14:20      2026-01-13 14:30             │
│                                                                             │
│  ─────────────────────────────────────────────────────────────────────────  │
│  PERFORMANCE                                                                │
│  ─────────────────────────────────────────────────────────────────────────  │
│                                                                             │
│  Duration:              2m 45s                3m 20s        (-35s ↓)       │
│  Cost:                  $0.12                 $0.15         (-$0.03 ↓)     │
│  Sessions:              5                     7             (-2 ↓)         │
│  Tokens:                6,800                 8,200         (-1,400 ↓)     │
│                                                                             │
│  ─────────────────────────────────────────────────────────────────────────  │
│  INPUTS                                                                     │
│  ─────────────────────────────────────────────────────────────────────────  │
│                                                                             │
│  topic:                 "machine learning"    "machine learning"   (same)  │
│  depth:                 "deep"                "comprehensive"      (diff)  │
│  max_sources:           10                    15                   (diff)  │
│                                                                             │
│  ─────────────────────────────────────────────────────────────────────────  │
│  OUTPUTS                                                                    │
│  ─────────────────────────────────────────────────────────────────────────  │
│                                                                             │
│  confidence:            0.92                  0.88                 (diff)  │
│  sources_count:         8                     12                   (diff)  │
│                                                                             │
│  ─────────────────────────────────────────────────────────────────────────  │
│  SUMMARY                                                                    │
│  ─────────────────────────────────────────────────────────────────────────  │
│                                                                             │
│  #1 was 18% faster and 20% cheaper than #5.                                │
│  #1 had higher confidence (0.92 vs 0.88) despite fewer sources.            │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Execution Behavior

When the `/cheesecake history` command is invoked:

### 1. Load History Index

```
Check for .cheesecake/history/index.json
IF EXISTS:
  Load index for fast listing
ELSE:
  Scan .cheesecake/history/*.json files
  Build index
```

### 2. Apply Filters

```
FOR each filter option:
  Apply filter to execution list
END FOR
```

### 3. Display Results

```
IF --json:
  Output JSON format
ELIF --compact:
  Output compact format
ELSE:
  Output table format
END IF
```

### 4. Handle Detail View

```
IF #N specified:
  Load full execution record
  Display detailed view
END IF
```

### 5. Handle Cleanup

```
IF --clear:
  IF --dry-run:
    Show what would be deleted
  ELSE:
    Confirm with user
    Delete matching entries
    Update index
  END IF
END IF
```

---

## Options Reference

| Option | Description | Example |
|--------|-------------|---------|
| `#N` | Show execution number N | `#1`, `#42` |
| `--all` | Show all executions | |
| `--limit N` | Limit to N entries (default: 10) | `--limit 20` |
| `--status S` | Filter by status | `--status failed` |
| `--program P` | Filter by program name | `--program workflow` |
| `--since DATE` | Filter after date | `--since 2026-01-01` |
| `--before DATE` | Filter before date | `--before 2026-01-15` |
| `--cost-above N` | Filter by min cost | `--cost-above 0.50` |
| `--cost-below N` | Filter by max cost | `--cost-below 0.10` |
| `--tag T` | Filter by tag | `--tag production` |
| `--json` | Output as JSON | |
| `--compact` | Compact output | |
| `--stats` | Show statistics | |
| `--compare #A #B` | Compare executions | `--compare #1 #5` |
| `--clear` | Clear history | |
| `--dry-run` | Preview clear operation | |

---

## Error Messages

### No History Found

```
No execution history found.

Run a CheeseCake program first:
  /cheesecake run your-program.cheesecake
```

### Execution Not Found

```
Execution #99 not found.

Available executions: #1 to #47

Use /cheesecake history to see all executions.
```

### Invalid Filter

```
Invalid date format: "jan 1"

Expected format: YYYY-MM-DD
Example: --since 2026-01-01
```

---

## Related Commands

| Command | Purpose |
|---------|---------|
| `/cheesecake replay` | Replay a previous execution |
| `/cheesecake run` | Execute a CheeseCake program |
| `/cheesecake` | Main CheeseCake menu |
