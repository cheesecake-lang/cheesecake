---
description: Validate .cheesecake file syntax without executing
allowed-tools: Read
---

# Usage

```
/cheesecake validate <filename>
```

# What It Does

1. Reads the .cheesecake file
2. Parses syntax against SKILL.md specification
3. Checks:
   - All keywords are valid
   - Agent definitions are well-formed
   - Variables declared before use
   - Loops have MAX limits
   - Brackets are balanced
4. Reports errors with line numbers and suggestions
5. Shows ✓ if valid

# Output

**Valid**:
```
✅ workflow.cheesecake is valid!

✓ Syntax correct
✓ 3 agents defined
✓ All variables declared
✓ Loops have limits

Ready to run: /cheesecake run workflow.cheesecake
```

**Invalid**:
```
❌ Validation Failed: workflow.cheesecake

Line 15: Agent 'researcher' not defined
Line 23: Loop missing MAX limit

Fix these issues before running.
```
