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
/cheesecake run <filename>
```

or

```
/cheesecake run
```
(will prompt for file selection)

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

## Special Flags (Future v0.0.2)

These will be implemented in future versions:

```
/cheesecake run --dry-run <file>    # Preview without executing
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
