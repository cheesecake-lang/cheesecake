# Module 11: Cost Management - IMPLEMENTATION PLAN

**Module**: Cost Management (v0.0.2)
**Status**: In Progress
**Started**: 2026-01-07

---

## Overview

Module 11 adds comprehensive cost management to CheeseCake, enabling users to:
- Set budget limits and cost thresholds
- Track costs in real-time during execution
- Get warnings before expensive operations
- Enforce budget limits automatically
- Optimize workflows based on cost analysis

---

## Core Features

### 1. CONFIG Block
Global configuration for cost management and execution settings.

**Syntax**:
```cheesecake
CONFIG:
  # Cost Management
  BUDGET: $1.00
  CONFIRM_COST_ABOVE: $0.10
  WARN_PARALLEL_ABOVE: 5

  # Model Defaults
  DEFAULT_MODEL: sonnet

  # Execution Settings
  MAX_PARALLEL_SESSIONS: 10
  TIMEOUT_DEFAULT: 120s
END CONFIG
```

**Settings**:
- `BUDGET`: Maximum total cost for workflow ($)
- `CONFIRM_COST_ABOVE`: Ask user before operations above this cost
- `WARN_PARALLEL_ABOVE`: Warn when spawning N+ parallel sessions
- `DEFAULT_MODEL`: Default model when not specified
- `MAX_PARALLEL_SESSIONS`: Hard limit on concurrent sessions
- `TIMEOUT_DEFAULT`: Default timeout for sessions

---

### 2. Cost Tracking

**Real-time tracking**:
- Track cost of each session execution
- Accumulate total cost during workflow
- Show running cost in progress display
- Compare against budget limit

**Cost display**:
```
Progress: [■■■■■■□□□□] 60% complete
Cost: $0.12 / $1.00 budget (12% used)
```

---

### 3. Cost Warnings

**Automatic warnings before expensive operations**:

```cheesecake
# Example: Large PARALLEL block
PARALLEL:
  VAR r1 = RUN SESSION(opus_agent): TASK: "..."
  VAR r2 = RUN SESSION(opus_agent): TASK: "..."
  VAR r3 = RUN SESSION(opus_agent): TASK: "..."
  # ... 7 more sessions
END PARALLEL

# VM shows warning:
# ⚠️  COST WARNING
# This PARALLEL block will spawn 10 Opus sessions
# Estimated cost: ~$0.30
# Current total: $0.12 / $1.00 budget
# Continue? [Y/n]
```

**Warning triggers**:
- Single operation exceeds `CONFIRM_COST_ABOVE`
- PARALLEL block spawns more than `WARN_PARALLEL_ABOVE` sessions
- Current cost + estimated operation exceeds `BUDGET`
- Expensive model (Opus) used in loop

---

### 4. Budget Enforcement

**Hard budget limit**:
```cheesecake
CONFIG:
  BUDGET: $1.00
END CONFIG

# If workflow tries to exceed budget:
# ❌ BUDGET EXCEEDED
# Current cost: $0.95
# Next operation estimated: $0.15
# Total would be: $1.10 (exceeds $1.00 budget)
# Workflow stopped.
```

**Soft budget warning**:
```cheesecake
CONFIG:
  BUDGET: $1.00
  WARN_AT_PERCENT: 80
END CONFIG

# At 80% budget:
# ⚠️  BUDGET WARNING
# Current cost: $0.80 / $1.00 (80%)
# Estimated remaining operations: $0.25
# You may exceed budget. Continue? [Y/n]
```

---

### 5. Cost Optimization Suggestions

**VM provides suggestions**:
```
💡 OPTIMIZATION SUGGESTION
Loop at line 45 uses Opus model (6 iterations, ~$0.48)
Consider using Sonnet instead to save ~$0.38 (79% reduction)
```

**Common suggestions**:
- Switch from Opus to Sonnet for simple tasks
- Use PARALLEL for independent operations
- Add MAX limits to potentially long loops
- Cache results with CHECKPOINT
- Use CHOICE ON to skip unnecessary work

---

## Files to Create/Modify

### 1. skills/cheesecake/cost-management.md (NEW)
**Purpose**: Complete specification for cost management features
**Size**: ~1,000 lines
**Sections**:
- CONFIG block syntax
- Cost tracking mechanics
- Warning system
- Budget enforcement
- Optimization suggestions
- Integration with estimation (Module 9)
- Examples and best practices

---

### 2. skills/cheesecake/SKILL.md (UPDATE)
**Purpose**: Add CONFIG construct to language spec
**Changes**:
- Add new section: "11. Configuration (CONFIG Block)"
- Document all CONFIG settings
- Show examples of cost management
- Explain interaction with other constructs
**Addition**: ~300 lines

---

### 3. skills/cheesecake/vm.md (UPDATE)
**Purpose**: Define VM behavior for cost tracking and warnings
**Changes**:
- Add cost tracking protocol
- Define warning trigger logic
- Specify budget enforcement behavior
- Integration with progress tracking
**Addition**: ~400 lines

---

### 4. test-cost-management.cheesecake (NEW)
**Purpose**: Test all cost management features
**Size**: ~200 lines
**Test scenarios**:
- CONFIG block parsing
- Cost tracking accuracy
- Warning triggers
- Budget enforcement
- Optimization suggestions

---

### 5. CHANGELOG.md (UPDATE)
**Purpose**: Document Module 11 additions
**Changes**: Add Module 11 section under v0.0.2

---

## Implementation Steps

### Step 1: Create Cost Management Specification ✅
**File**: `skills/cheesecake/cost-management.md`
**Goal**: Complete specification of all cost management features

**Content**:
1. CONFIG block syntax and semantics
2. Cost tracking mechanics
3. Warning system (triggers and format)
4. Budget enforcement (hard and soft limits)
5. Optimization suggestions
6. Integration with Module 9 (cost estimation)
7. 5+ comprehensive examples
8. Best practices
9. Testing guidelines

---

### Step 2: Update SKILL.md with CONFIG ⏳
**File**: `skills/cheesecake/SKILL.md`
**Goal**: Add CONFIG construct to language specification

**Changes**:
1. Add new section 11: "Configuration (CONFIG Block)"
2. Document syntax and all settings
3. Show examples of CONFIG usage
4. Explain precedence and scope
5. Show interaction with other constructs
6. Add to table of contents

---

### Step 3: Update VM with Cost Tracking ⏳
**File**: `skills/cheesecake/vm.md`
**Goal**: Define VM execution behavior for cost management

**Changes**:
1. Add cost tracking protocol
2. Define cost calculation per operation
3. Specify warning display format
4. Define budget check points
5. Explain user confirmation flow
6. Integration with progress display

---

### Step 4: Create Test File ⏳
**File**: `test-cost-management.cheesecake`
**Goal**: Comprehensive test of cost management features

**Tests**:
1. CONFIG block parsing
2. Budget enforcement (exceed limit)
3. Warning trigger (expensive operation)
4. Cost tracking accuracy
5. Parallel session warning
6. Optimization suggestions
7. Multiple CONFIG scenarios

---

### Step 5: Update CHANGELOG ⏳
**File**: `CHANGELOG.md`
**Goal**: Document Module 11 completion

**Addition**: Module 11 section with features list

---

### Step 6: Testing & Validation ⏳
**Goal**: Ensure all features work correctly

**Tasks**:
1. Read all modified files
2. Validate syntax consistency
3. Check cross-references
4. Verify examples are complete
5. Test cost calculation logic

---

### Step 7: Final Commit ⏳
**Goal**: Commit Module 11 to repository

**Commit message**: "Complete Module 11: Cost Management"

---

## Success Criteria

- [x] CONFIG block syntax defined
- [ ] Cost tracking mechanism specified
- [ ] Warning system designed
- [ ] Budget enforcement rules clear
- [ ] Optimization suggestions documented
- [ ] All files created/updated
- [ ] Test file comprehensive
- [ ] Documentation complete
- [ ] Examples working
- [ ] Backward compatible with v0.0.1 and v0.0.2 modules

---

## Integration Points

### With Module 9 (Progress & Cost Estimation)
- Cost tracking uses estimation formulas
- Real-time cost updates progress display
- Dry-run shows costs without tracking

### With Other Constructs
- CONFIG applies globally to entire workflow
- Cost warnings appear during PARALLEL, LOOP, SESSION
- Budget checks at each operation
- Optimization suggestions based on code analysis

---

## Backward Compatibility

✅ **All v0.0.1 and v0.0.2 workflows continue to work**
- CONFIG block is optional
- Default: no budget limit, no warnings
- Existing workflows without CONFIG run normally
- No breaking changes to syntax

---

## Examples Preview

### Example 1: Basic Budget Control
```cheesecake
CONFIG:
  BUDGET: $0.50
  CONFIRM_COST_ABOVE: $0.10
END CONFIG

AGENT Researcher:
  MODEL: opus
  PROMPT: "Research agent"

VAR researcher = NEW Researcher()

# This will trigger warning (Opus session ~$0.15)
VAR result = RUN SESSION(researcher):
  TASK: "Comprehensive research on AI safety"

# VM shows:
# ⚠️  COST WARNING
# This Opus session estimated at: ~$0.15
# Current budget: $0.50 (30% would be used)
# Continue? [Y/n]
```

### Example 2: Parallel Session Warning
```cheesecake
CONFIG:
  WARN_PARALLEL_ABOVE: 5
  BUDGET: $1.00
END CONFIG

AGENT Worker:
  MODEL: sonnet
  PROMPT: "Worker agent"

VAR items = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# This triggers warning (10 parallel sessions)
PARALLEL FOR item IN items:
  VAR result = RUN SESSION(worker):
    TASK: "Process {item}"
END PARALLEL FOR

# VM shows:
# ⚠️  PARALLEL WARNING
# About to spawn 10 parallel sessions
# Estimated cost: ~$0.30
# Current: $0.00 / $1.00 budget
# Continue? [Y/n]
```

### Example 3: Budget Exceeded
```cheesecake
CONFIG:
  BUDGET: $0.20
END CONFIG

AGENT OpusAgent:
  MODEL: opus
  PROMPT: "High-quality agent"

VAR agent = NEW OpusAgent()

# First session succeeds (~$0.15)
VAR result1 = RUN SESSION(agent):
  TASK: "Task 1"

# Second session blocked
VAR result2 = RUN SESSION(agent):
  TASK: "Task 2"

# VM shows:
# ❌ BUDGET EXCEEDED
# Current cost: $0.15
# Next operation estimated: ~$0.15
# Total would be: $0.30 (exceeds $0.20 budget)
# Workflow stopped at line 18
```

### Example 4: Optimization Suggestion
```cheesecake
CONFIG:
  BUDGET: $1.00
END CONFIG

AGENT Writer:
  MODEL: opus  # Expensive!
  PROMPT: "Writer"

VAR writer = NEW Writer()

# Loop with expensive model
LOOP UNTIL **{draft} is ready** MAX 10:
  VAR draft = RUN SESSION(writer):
    TASK: "Improve draft"
END LOOP

# VM suggests:
# 💡 OPTIMIZATION SUGGESTION
# Loop at line 14 uses Opus model (up to 10 iterations)
# Estimated cost: ~$0.75 - $1.50
# Consider using Sonnet to save ~$0.60 (80% reduction)
# Quality impact: Minimal for iterative refinement
```

---

## Next Steps

1. ✅ Create this plan document
2. ⏳ Create `skills/cheesecake/cost-management.md`
3. ⏳ Update `skills/cheesecake/SKILL.md`
4. ⏳ Update `skills/cheesecake/vm.md`
5. ⏳ Create `test-cost-management.cheesecake`
6. ⏳ Update `CHANGELOG.md`
7. ⏳ Final testing and validation
8. ⏳ Commit and push

**Let's start with Step 1!** 🚀
