# CheeseCake Main Command
# Purpose: Entry point for CheeseCake - shows menu and detects files
# Part of: CheeseCake v0.0.1 - Module 4 (Commands)
#
# This command is invoked when user types `/cheesecake` without arguments.
# It scans for .cheesecake files and presents an interactive menu.
#
# Usage:
#   /cheesecake
#
# Dependencies:
#   - Glob tool (for finding .cheesecake files)
#   - Other cheesecake commands for delegation

---
description: CheeseCake main menu - detect files and show options
---

# CheeseCake Main Menu

When the user invokes `/cheesecake`, you should:

## Step 1: Scan for .cheesecake Files

Use the Glob tool to find all `.cheesecake` files in the current directory and subdirectories:

```
Glob: **/*.cheesecake
```

Store the list of found files.

## Step 2: Display Welcome & Menu

Show a welcome message and interactive menu:

```
╔═══════════════════════════════════════════════════════════╗
║                                                           ║
║              🧀 Welcome to CheeseCake v0.0.1              ║
║     Structured OOP Language for AI Agent Orchestration    ║
║                                                           ║
╚═══════════════════════════════════════════════════════════╝

[Found .cheesecake files section if files exist]

[Menu options]

[Help footer]
```

### If Files Found

```
📂 Found X .cheesecake file(s):

   1. workflow.cheesecake
   2. research-pipeline.cheesecake
   3. templates/hello-world.cheesecake

────────────────────────────────────────────────────────────

What would you like to do?

  [1] 🚀 Run existing file
  [2] ✨ Create new file (with Helper)
  [3] 📝 Create new file (from scratch)
  [4] 📚 Create from template
  [5] ✅ Validate a file
  [6] 📖 Explain a file
  [7] ❓ Get help

────────────────────────────────────────────────────────────
💡 Quick commands:
   /cheesecake run <file>     - Execute a workflow
   /cheesecake create         - Create with Helper
   /cheesecake help           - View documentation

Type a number (1-7) or use a command above.
```

### If No Files Found

```
📂 No .cheesecake files found in this project.

────────────────────────────────────────────────────────────

Let's get started!

  [1] ✨ Create new file (with Helper)
      Describe what you want in plain English,
      and I'll generate a .cheesecake file for you.

  [2] 📝 Create new file (from scratch)
      I'll create a blank file with helpful comments.

  [3] 📚 Create from template
      Start with a pre-built example (research, code-review, etc.)

  [4] 📖 View examples
      See example .cheesecake files to learn the syntax.

  [5] ❓ Get help
      Learn about CheeseCake and its features.

────────────────────────────────────────────────────────────
💡 New to CheeseCake? Try option [1] - the Helper will guide you!

Type a number (1-5) or run: /cheesecake help
```

## Step 3: Handle User Response

Based on the user's choice, delegate to appropriate command:

### Option 1: Run File

If files exist, ask which file to run, then invoke `/cheesecake run <filename>`

If only one file, offer to run it directly:
```
Run workflow.cheesecake? [Y/n]
```

### Option 2: Create with Helper

Invoke `/cheesecake create` (helper wizard)

### Option 3: Create from Scratch

Create a blank template file and open it for editing:

```
Created: my-workflow.cheesecake

I've created a blank workflow with helpful comments.
Edit the file to add your workflow logic.

See /cheesecake help for syntax reference.
```

### Option 4: Create from Template

Show template options:
```
Available templates:

  1. hello-world        - Simplest example
  2. research-and-write - Research workflow → article
  3. code-review        - Review code changes
  4. data-pipeline      - Process data through stages
  5. content-moderation - Review and moderate content

Which template? (1-5)
```

Then copy chosen template and let user customize it.

### Option 5: Validate File

Ask which file to validate, then invoke `/cheesecake validate <filename>`

### Option 6: Explain File

Ask which file to explain, then invoke `/cheesecake explain <filename>`

### Option 7: Get Help

Invoke `/cheesecake help`

## Step 4: Continuous Interaction

After completing an action, offer to:
- Run another command
- Return to main menu
- Exit

## Error Handling

If glob fails or other errors occur:
```
⚠️ Error: [error message]

You can still use CheeseCake commands manually:
  /cheesecake run <file>
  /cheesecake create
  /cheesecake help
```

## Examples

### Example 1: User types `/cheesecake`

**You respond with**:
```
╔═══════════════════════════════════════════════════════════╗
║              🧀 Welcome to CheeseCake v0.0.1              ║
║     Structured OOP Language for AI Agent Orchestration    ║
╚═══════════════════════════════════════════════════════════╝

📂 Found 2 .cheesecake file(s):

   1. research-workflow.cheesecake
   2. hello.cheesecake

────────────────────────────────────────────────────────────

What would you like to do?

  [1] 🚀 Run existing file
  [2] ✨ Create new file (with Helper)
  [3] 📝 Create new file (from scratch)
  [4] 📚 Create from template
  [5] ✅ Validate a file
  [6] 📖 Explain a file
  [7] ❓ Get help

Type a number or use quick commands like /cheesecake run <file>
```

### Example 2: User chooses option 1 (Run)

**You respond**:
```
Which file would you like to run?

  1. research-workflow.cheesecake
  2. hello.cheesecake

Enter number (1-2):
```

User says "1", you execute:
```
Running: research-workflow.cheesecake
...
[execution happens via /cheesecake run]
```

## Summary

The main `/cheesecake` command is the **user-friendly entry point** that:
- Discovers existing workflows
- Guides new users
- Provides quick access to all features
- Makes CheeseCake approachable

Always be helpful, clear, and guide users toward success!
