# CheeseCake Syntax Reference
# Purpose: Quick reference guide for CheeseCake syntax
# Part of: CheeseCake v0.0.1 - Module 2 (Language Specification)
#
# This file provides a condensed syntax reference for quick lookup.
# For detailed explanations, see SKILL.md
#
# Usage:
#   Quick reference when writing .cheesecake files
#
# Related:
#   - SKILL.md (complete specification)

---

# CheeseCake Syntax Quick Reference

## Comments

```cheesecake
# Single line comment
```

## Skills (Traits)

```cheesecake
SKILL skill_name:
  CAPABILITIES: [cap1, cap2]
  REQUIRES: [req1, req2]

SKILL child EXTENDS parent:
  CAPABILITIES: [additional]
```

## Agents (Classes)

```cheesecake
AGENT AgentName:
  MODEL: sonnet | opus | haiku
  SKILLS: [skill1, skill2]
  PROMPT: "Instructions"

AGENT Child EXTENDS Parent:
  MODEL: opus
  SKILLS: [+added_skill]
  PROMPT: PARENT + "Additional"

AGENT Multi IMPLEMENTS [skill1, skill2]:
  MODEL: sonnet
  PROMPT: "..."
```

## Variables

```cheesecake
VAR name = value          # Mutable
CONST name = value        # Immutable
name = new_value          # Reassignment (VAR only)
```

## Creating Agents & Sessions

```cheesecake
# Create agent instance
VAR agent = NEW AgentName()

# Create session
VAR session = SESSION(agent):
  TASK: "Natural language task"
  INPUT: {data}
  CONTEXT: {metadata}
  TIMEOUT: 30s
  RETRY: 3

# Run session
VAR result = RUN session

# Inline
VAR result = RUN SESSION(agent):
  TASK: "Do something"
  INPUT: {data}
```

## Control Flow

### Sequential

```cheesecake
SEQUENCE:
  VAR step1 = ...
  VAR step2 = ...
END SEQUENCE
```

### Parallel

```cheesecake
PARALLEL:
  VAR r1 = RUN SESSION(a1): TASK: "Task 1"
  VAR r2 = RUN SESSION(a2): TASK: "Task 2"
END PARALLEL

PARALLEL JOIN(first):  # Race mode
  VAR fast = ...
  VAR slow = ...
END PARALLEL

PARALLEL JOIN(any, N):  # Wait for N
  ...
END PARALLEL
```

### Conditionals

```cheesecake
IF **{var} meets condition**:
  ...
ELIF **{var} meets other**:
  ...
ELSE:
  ...
END IF

CHOICE ON **criteria**:
  OPTION "label1":
    ...
  OPTION "label2":
    ...
END CHOICE
```

## Loops

### Fixed

```cheesecake
REPEAT count:
  ...
END REPEAT

REPEAT count AS index:
  ...
END REPEAT
```

### For-Each

```cheesecake
FOR item IN list:
  ...
END FOR

FOR item, index IN list:
  ...
END FOR

PARALLEL FOR item IN list:
  ...
END PARALLEL FOR
```

### Unbounded

```cheesecake
LOOP UNTIL **{var} condition** MAX limit:
  ...
END LOOP

WHILE **{var} condition** MAX limit:
  ...
END WHILE

LOOP MAX limit:
  ...
END LOOP
```

## State & Persistence

### Checkpoints

```cheesecake
CHECKPOINT "name":
  SAVE: {var1, var2}
  TO: "path"
END CHECKPOINT

IF CHECKPOINT_EXISTS("name"):
  RESTORE FROM "name"
ELSE:
  ...
END IF
```

### Memory

```cheesecake
MEMORY name:
  key: "value"
  list: []
END MEMORY

MEMORY name.key = "new"
MEMORY name.list.APPEND(item)
VAR data = LOAD MEMORY name
```

## Error Handling

```cheesecake
TRY:
  VAR result = ...
CATCH error:
  LOG ERROR: "{error.message}"
FINALLY:
  CLEANUP resources
END TRY

# With retry
VAR data = RUN SESSION(agent):
  TASK: "..."
  RETRY: 3
  BACKOFF: exponential
  ON_FAIL: THROW "message"
```

## Functions

```cheesecake
FUNCTION name(param1, param2):
  VAR result = ...
  RETURN result
END FUNCTION

VAR output = CALL name(arg1, arg2)
```

## Modules

```cheesecake
# Import
IMPORT "path/file.cheesecake" AS alias
VAR a = NEW alias.Agent()

# Export
EXPORT AGENT MyAgent
EXPORT SKILL MySkill
EXPORT FUNCTION my_func
```

## Built-in Functions

### Logging

```cheesecake
LOG "message"
LOG INFO: "message"
LOG WARNING: "message"
LOG ERROR: "message"
LOG SUCCESS: "message"
```

### Output

```cheesecake
PRINT value
SAVE value TO "path"
```

### Data

```cheesecake
LOAD "path"
FIRST(list)
LAST(list)
LENGTH(list)
REMOVE(list, item)
```

### Time

```cheesecake
NOW()
```

### Utility

```cheesecake
CLEANUP resource
CREATE_TEMP_FILE()
DELETE file
```

## Semantic Conditions

Use `**...**` for AI-evaluated conditions:

```cheesecake
**{var} is greater than 100**
**{status} equals "complete"**
**{draft} is ready for publication**
**{findings} shows significant progress**
**{list} contains item X**
**{a} is true AND {b} is false**
```

## Variable Interpolation

Use `{var}` in strings:

```cheesecake
TASK: "Research {topic} and provide {depth} analysis"
LOG "Completed iteration {index} of {total}"
```

## Keywords Reference

| Keyword | Purpose |
|---------|---------|
| `SKILL` | Define capability bundle |
| `AGENT` | Define agent class |
| `VAR` | Declare mutable variable |
| `CONST` | Declare immutable variable |
| `NEW` | Create agent instance |
| `SESSION` | Create session object |
| `RUN` | Execute session |
| `TASK` | Task description |
| `INPUT` | Input data |
| `CONTEXT` | Additional context |
| `EXTENDS` | Inheritance |
| `IMPLEMENTS` | Composition |
| `SEQUENCE` | Sequential block |
| `PARALLEL` | Parallel block |
| `IF/ELIF/ELSE` | Conditional |
| `CHOICE/OPTION` | AI-selected branch |
| `REPEAT` | Fixed loop |
| `FOR` | For-each loop |
| `LOOP` | Unbounded loop |
| `UNTIL` | Loop exit condition |
| `WHILE` | Loop continue condition |
| `MAX` | Maximum iterations |
| `CHECKPOINT` | Save state |
| `RESTORE` | Load state |
| `MEMORY` | Persistent memory |
| `TRY/CATCH/FINALLY` | Error handling |
| `THROW` | Raise error |
| `RETRY` | Retry count |
| `BACKOFF` | Retry strategy |
| `FUNCTION` | Define function |
| `RETURN` | Return value |
| `CALL` | Call function |
| `IMPORT` | Import module |
| `EXPORT` | Export definitions |
| `AS` | Alias |
| `LOG` | Log message |
| `PRINT` | Print output |
| `SAVE` | Save to file |
| `LOAD` | Load from file/memory |
| `END` | Close block |

## Complete Minimal Example

```cheesecake
# Define agent
AGENT Researcher:
  MODEL: sonnet
  PROMPT: "You are a researcher."

# Create and run
VAR r = NEW Researcher()
VAR result = RUN SESSION(r): TASK: "Research AI"
PRINT result
```

## Complete Full Example

```cheesecake
# Import
IMPORT "common/agents.cheesecake" AS lib

# Define
SKILL analysis:
  CAPABILITIES: [data-analysis]

AGENT Analyst:
  MODEL: sonnet
  SKILLS: [analysis]
  PROMPT: "You analyze data."

# Function
FUNCTION analyze(data):
  VAR agent = NEW Analyst()
  VAR result = RUN SESSION(agent):
    TASK: "Analyze {data}"
  RETURN result
END FUNCTION

# Main workflow
VAR data = LOAD "data.json"
VAR analysis = CALL analyze(data)

IF **{analysis} shows issues**:
  LOG WARNING: "Issues found"
ELSE:
  LOG SUCCESS: "All good"
END IF

SAVE analysis TO "output.json"
```

---

For complete documentation, see [SKILL.md](SKILL.md)
