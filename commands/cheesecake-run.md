# CheeseCake Run Command
# Purpose: Execute .cheesecake workflow files
# Part of: CheeseCake v0.0.1 - Module 4 (Commands)
#
# This command executes a .cheesecake file by invoking the CheeseCake VM agent.
#
# Usage:
#   /cheesecake run <filename>
#   /cheesecake run (interactive file selection)
#
# Dependencies:
#   - Read tool (to read .cheesecake file)
#   - Task tool (to invoke VM agent)
#   - agents/vm/AGENT.md (CheeseCake VM)

---
description: Execute a .cheesecake workflow file
allowed-tools: Read, Task, Glob
---

# CheeseCake Run Command

Execute a `.cheesecake` workflow file.

## Usage

```
/cheesecake run <filename> [--dry-run] [--verbose]
```

or

```
/cheesecake run
```
(will prompt for file selection)

### Flags

- `--dry-run`: Simulate execution without spawning agents or incurring costs
- `--verbose`: Show detailed progress and token tracking (v0.0.2+)

## Execution Protocol

When user invokes `/cheesecake run <filename>`:

### Step 1: Locate File

1. If filename provided:
   - Check if file exists
   - If not found, check with `.cheesecake` extension added
   - If still not found, show error and list available files

2. If no filename provided:
   - Use Glob to find all `**/*.cheesecake` files
   - Show list and ask user to select

### Step 2: Read File

Use the Read tool to load the .cheesecake file contents:

```
Read: path/to/workflow.cheesecake
```

### Step 3: Display Execution Banner

```
╔═══════════════════════════════════════════════════════════╗
║           🚀 Executing CheeseCake Workflow                ║
╚═══════════════════════════════════════════════════════════╝

📄 File: workflow.cheesecake
📝 Lines: X
⏱️  Started: [timestamp]

────────────────────────────────────────────────────────────
```

### Step 4: Invoke CheeseCake VM

Use the Task tool to spawn the CheeseCake VM agent:

```
Task(
  agent: "cheesecake-vm",
  prompt: "Execute this .cheesecake workflow:\n\n```cheesecake\n[file contents]\n```\n\nFollow the execution protocol defined in your agent instructions.",
  context: {
    filename: "workflow.cheesecake",
    working_directory: "[current dir]"
  }
)
```

The VM agent will:
1. Parse the file
2. Validate syntax
3. Execute statements
4. Spawn sub-agent sessions
5. Report progress
6. Return results

### Step 5: Display Results

Show the VM's execution output, including:
- Progress updates
- Any output from PRINT statements
- Success/error status
- Execution time

```
────────────────────────────────────────────────────────────

✅ Execution Complete!

⏱️  Duration: Xm Ys
📊 Sessions spawned: N
💬 Output:

[Any PRINT output or final results]

────────────────────────────────────────────────────────────
```

### Step 6: Post-Execution Actions

Offer next actions:
```
What would you like to do next?

  [1] Run again
  [2] Run different file
  [3] Explain this workflow
  [4] Return to main menu

Or use: /cheesecake [command]
```

## Error Handling

### File Not Found

```
❌ Error: File not found: workflow.cheesecake

Available .cheesecake files:
  • hello.cheesecake
  • research-pipeline.cheesecake

Run with: /cheesecake run <filename>
Or: /cheesecake (for menu)
```

### Syntax Error

If the VM reports a syntax error:
```
❌ Syntax Error in workflow.cheesecake

Line 15: Agent 'researcher' not defined

  13: VAR writer = NEW Writer()
  14:
> 15: VAR result = RUN SESSION(researcher): TASK: "Research"
  16:

💡 Suggestions:
  1. Did you mean 'Researcher'? (case-sensitive)
  2. Define the agent first:
     AGENT researcher:
       MODEL: sonnet
  3. Import from common agents:
     IMPORT "agents.cheesecake" AS lib

Fix the error and run again: /cheesecake run workflow.cheesecake
```

### Execution Error

If execution fails during runtime:
```
❌ Execution Error

At line 25: Session failed - timeout after 30s

The session for agent 'researcher' timed out while executing:
  Task: "Research comprehensive quantum computing advances"

💡 Suggestions:
  1. Increase timeout: TIMEOUT: 60s
  2. Break into smaller tasks
  3. Check if external API is available

Stack trace:
  Line 20: VAR findings = RUN SESSION...
  Line 25: ... (failure point)
```

## Dry-Run Mode (v0.0.2+)

### Overview

Dry-run mode simulates workflow execution WITHOUT actually spawning agents or incurring costs.

### Usage

```
/cheesecake run workflow.cheesecake --dry-run
```

### What Dry-Run Does

1. **Parses the file** - Validates syntax
2. **Simulates execution** - Walks through logic
3. **Calculates costs** - Estimates tokens and expenses
4. **Shows what WOULD happen** - Full preview
5. **NO actual execution** - Zero cost, zero API calls

### Dry-Run Execution Protocol

When `--dry-run` flag is present:

#### Step 1: Parse & Validate

- Read the .cheesecake file
- Validate syntax against SKILL.md
- Check for undefined agents, invalid constructs
- Report any syntax errors

#### Step 2: Build Execution Plan

- Identify all agents to be created
- Count SESSION executions
- Detect PARALLEL blocks
- Estimate loop iterations (use MAX for LOOP UNTIL)
- Map out phases if PHASE blocks exist

#### Step 3: Simulate Execution

Walk through each statement WITHOUT actual execution:

```
✓ Line 7: Define AGENT SentimentAnalyzer (Sonnet)
✓ Line 12: Create instance 'analyzer'
✓ Line 44: FOR loop start (10 iterations)
  → Would spawn 10 Sonnet sessions
  → Estimated: 8,000 tokens
  → Est. cost: ~$0.06
✓ Line 67: FOR loop end
✓ Line 134: SAVE to output/report.json
```

#### Step 4: Calculate Costs

Use cost-estimation.md formulas:
- Count sessions by model type
- Estimate tokens per session
- Calculate total cost
- Provide cost range (min/max)

#### Step 5: Display Summary

Show comprehensive dry-run report:

```
╔═══════════════════════════════════════════════════════════╗
║  DRY RUN: workflow.cheesecake                             ║
╚═══════════════════════════════════════════════════════════╝

Simulating execution (no actual sessions spawned)...

[■■■■■■■■■■] 100% simulated

📊 Analysis:
  • 1 agent defined (SentimentAnalyzer - Sonnet)
  • 10 total sessions (sequential)
  • 1 FOR loop (10 iterations)
  • 1 conditional block

─────────────────────────────────────────────────────────────
💰 COST ESTIMATE:

Sessions:
  • 10 Sonnet (sentiment analysis):   ~$0.06

Total estimated: $0.06
Range: $0.05 - $0.08

Tokens: ~8,000
Estimated time: ~15-20s

─────────────────────────────────────────────────────────────
📁 Output:
  • Would save to: output/feedback-analysis-report.json

─────────────────────────────────────────────────────────────
✅ Dry run complete - no costs incurred

Ready to run for real? Use:
  /cheesecake run workflow.cheesecake

Or estimate only:
  /cheesecake estimate workflow.cheesecake
─────────────────────────────────────────────────────────────
```

### Dry-Run with Phases

If workflow has PHASE blocks, show phase-by-phase:

```
╔═══════════════════════════════════════════════════════════╗
║  DRY RUN: research-workflow.cheesecake                    ║
╚═══════════════════════════════════════════════════════════╝

○ Phase 1: Research
  • Would create Researcher agent (Sonnet)
  • Would spawn 3 parallel sessions
  • Estimated: 7,500 tokens, ~$0.06, ~8s

○ Phase 2: Analysis
  • Would create Analyst agent (Sonnet)
  • Would run 1 analysis session
  • Estimated: 2,500 tokens, ~$0.02, ~3s

○ Phase 3: Writing
  • Would create Writer agent (Opus)
  • Would run 1 writing session
  • Would enter loop (max 5, est. 3 iterations)
  • Estimated: 15,000 tokens, ~$0.45, ~20s

○ Phase 4: Output
  • Would save to output/article.md
  • No cost (file I/O only), ~0.1s

─────────────────────────────────────────────────────────────
💰 TOTAL ESTIMATE:

Cost: $0.53 (range: $0.35 - $0.85)
Tokens: ~25,000
Time: ~31-45s

Cost breakdown:
  Research:   $0.06  (11%)
  Analysis:   $0.02  (4%)
  Writing:    $0.45  (85%)  ← Most expensive

─────────────────────────────────────────────────────────────
💡 Optimization suggestions:
  • Writing phase uses Opus (expensive)
  • Consider Sonnet for drafts, Opus for final polish
  • Potential savings: ~$0.30 (67%)

─────────────────────────────────────────────────────────────
Proceed with actual run? [Y/n]
```

### Dry-Run Implementation Notes

**For the executing agent:**

1. **Parse mode** - Use SKILL.md to validate syntax
2. **Simulation mode** - Don't spawn actual Task agents
3. **Cost calculation** - Use cost-estimation.md formulas
4. **Progress tracking** - Use progress.md display formats
5. **No side effects** - No file writes, no API calls, no state changes

**Key differences from normal execution:**

| Normal Run | Dry Run |
|------------|---------|
| Spawn Task agents | Simulate without spawning |
| Execute AI sessions | Estimate tokens/cost |
| Write files | Show "would write to..." |
| Update checkpoints | Show "would save checkpoint..." |
| Actual time varies | Instant simulation |
| Costs money | Zero cost |

### Verbose Mode (v0.0.2+)

Combine with `--verbose` for maximum detail:

```
/cheesecake run workflow.cheesecake --dry-run --verbose
```

Shows:
- Every statement execution
- Detailed token breakdown
- Model selection reasoning
- Full cost calculations
- Optimization opportunities

### Error Detection in Dry-Run

Dry-run catches errors WITHOUT cost:

```
╔═══════════════════════════════════════════════════════════╗
║  DRY RUN FAILED                                           ║
╚═══════════════════════════════════════════════════════════╝

❌ Syntax Error at line 48

  46: FOR feedback IN feedback_list:
  47:   VAR sentiment = RUN SESSION(analyzer): TASK: "..."
> 48:   IF {sentiment} is positive:
  49:     VAR positive_count = positive_count + 1

Error: Missing ** markers for semantic condition

Should be: IF **{sentiment} is positive**:

Fix this error and try again.

─────────────────────────────────────────────────────────────
✓ Dry run caught error before incurring any costs!
```

### Future Flags (v0.0.3+)

```
/cheesecake run --debug <file>      # Show detailed execution trace
/cheesecake run --resume <file>     # Resume from checkpoint
```

## Examples

### Example 1: Basic Execution

User: `/cheesecake run hello.cheesecake`

You:
```
╔═══════════════════════════════════════════════════════════╗
║           🚀 Executing CheeseCake Workflow                ║
╚═══════════════════════════════════════════════════════════╝

📄 File: hello.cheesecake
📝 Lines: 8
⏱️  Started: 14:23:15

────────────────────────────────────────────────────────────

[Invoke VM agent to execute]

✓ Parsed and validated
✓ Defined agent: Greeter
→ Running session: greeter
  Output: "Hello from CheeseCake! 🧀"

────────────────────────────────────────────────────────────

✅ Execution Complete!

⏱️  Duration: 2.3s
📊 Sessions spawned: 1
💬 Output: Hello from CheeseCake! 🧀
```

### Example 2: No File Specified

User: `/cheesecake run`

You:
```
Which workflow would you like to run?

📂 Available .cheesecake files:

  1. hello.cheesecake
  2. research-workflow.cheesecake
  3. templates/code-review.cheesecake

Enter number (1-3) or filename:
```

### Example 3: Complex Workflow

User: `/cheesecake run research-article.cheesecake`

You:
```
╔═══════════════════════════════════════════════════════════╗
║           🚀 Executing CheeseCake Workflow                ║
╚═══════════════════════════════════════════════════════════╝

📄 File: research-article.cheesecake
📝 Lines: 87
⏱️  Started: 14:25:00

────────────────────────────────────────────────────────────

✓ Parsed and validated
✓ Defined agents: Researcher, Analyst, Writer, Editor

→ Phase 1: Research (parallel)
  ⏳ Academic research...
  ⏳ Industry research...
  ⏳ Market analysis...

✓ Phase 1 complete (3 sessions, 8.5s)

✓ Checkpoint saved: research-complete

→ Phase 2: Synthesis
  ⏳ Synthesizing findings...

✓ Phase 2 complete (1 session, 3.2s)

→ Phase 3: Writing (iterative)
  ⏳ Writing first draft...
  ⏳ Editor review (iteration 1)...
  ⏳ Revision (iteration 1)...
  ⏳ Editor review (iteration 2)...
  ⏳ Revision (iteration 2)...
  ✓ Draft meets publication standards

✓ Phase 3 complete (5 sessions, 15.7s)

→ Phase 4: Final output
  ⏳ Preparing final version...

✓ Saved to: output/quantum-computing-article.md

────────────────────────────────────────────────────────────

✅ Execution Complete!

⏱️  Duration: 27.8s
📊 Sessions spawned: 9
📁 Output saved: output/quantum-computing-article.md

💬 Summary:
Article completed successfully!
Topic: quantum computing
Revisions: 2
Output: output/quantum-computing-article.md
```

## Best Practices

### 1. Clear Progress Reporting

Always show what's happening:
- What's being executed
- Current progress
- Results as they arrive

### 2. Actionable Errors

When errors occur:
- Show exactly where (line number)
- Explain what's wrong
- Suggest how to fix

### 3. Performance Metrics

Report:
- Total execution time
- Number of sessions spawned
- Any checkpoints created

### 4. User Guidance

After execution:
- Show what was produced
- Offer next steps
- Make it easy to iterate

## Summary

The `/cheesecake run` command is the **primary execution interface**. It:
- Loads and validates .cheesecake files
- Invokes the VM agent for execution
- Reports progress clearly
- Handles errors gracefully
- Guides users to success

Make execution smooth, transparent, and helpful!
