---
description: Initialize CheeseCake in current project
allowed-tools: Write, Bash
---

# Usage

```
/cheesecake init
```

# What It Does

1. Creates `.cheesecake/` directory structure:
   ```
   .cheesecake/
   ├── state/      # For checkpoints
   ├── memory/     # For persistent memory
   └── config.json # Project configuration
   ```

2. Creates `.gitignore` entry (if git repo):
   ```
   .cheesecake/state/
   .cheesecake/memory/
   ```

3. Creates `README.md` explaining the setup

# Output

```
✅ Initialized CheeseCake in this project!

Created:
  • .cheesecake/state/ (for checkpoints)
  • .cheesecake/memory/ (for persistent memory)
  • .cheesecake/config.json

Next steps:
  1. Create a workflow: /cheesecake create
  2. Or use a template: /cheesecake (option 4)

Ready to build AI workflows! 🧀
```
