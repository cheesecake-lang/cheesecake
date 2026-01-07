# Changelog

All notable changes to CheeseCake will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.0.1] - 2025-01-05

### Added

#### Core Language
- Object-oriented syntax with `AGENT` (classes) and `SKILL` (traits)
- Agent inheritance with `EXTENDS` keyword
- Agent composition with `IMPLEMENTS` keyword
- Variable declarations: `VAR` (mutable) and `CONST` (immutable)
- Session creation and execution: `SESSION()` and `RUN`

#### Control Flow
- Sequential execution with `SEQUENCE` blocks
- Parallel execution with `PARALLEL` blocks
- Join strategies: `JOIN(first)`, `JOIN(any, N)`
- Conditionals: `IF/ELIF/ELSE`
- AI-guided choice: `CHOICE ON` with `OPTION` branches
- Semantic conditions using `**...**` syntax

#### Loops
- Fixed iteration: `REPEAT N`
- For-each loops: `FOR item IN list`
- Parallel for-each: `PARALLEL FOR`
- Semantic exit loops: `LOOP UNTIL **condition**`
- While loops: `WHILE **condition**`
- All loops require `MAX` limit for safety

#### State Management
- Checkpoints for resumability: `CHECKPOINT` and `RESTORE`
- Persistent memory across sessions: `MEMORY` blocks
- File-based state persistence in `.cheesecake/state/`

#### Advanced Features
- Reusable functions: `FUNCTION` with `RETURN`
- Module system: `IMPORT` and `EXPORT`
- Error handling: `TRY/CATCH/FINALLY`
- Retry logic with backoff strategies
- Pipeline operations (planned for future)

#### Commands
- `/cheesecake` - Interactive main menu
- `/cheesecake run <file>` - Execute workflows
- `/cheesecake validate <file>` - Validate syntax
- `/cheesecake init` - Initialize project structure
- `/cheesecake help [topic]` - Get help
- `/cheesecake explain <file>` - Explain workflow
- `/cheesecake create` - Helper wizard with full AI-powered assistance

#### Templates
- `hello-world.cheesecake` - Simplest example
- `research-and-write.cheesecake` - Research to article workflow
- `code-review.cheesecake` - Automated code review
- `data-pipeline.cheesecake` - Data processing pipeline
- `content-moderation.cheesecake` - Content moderation workflow

#### Helper Agent (Zero Learning Curve)
- Natural language to CheeseCake code conversion
- Interactive workflow creation wizard
- Automatic code generation with explanations
- Iterative refinement based on user feedback
- Template customization support

#### Testing Framework
- `test-syntax.cheesecake` - Comprehensive syntax validation (20+ tests)
- `test-agents.cheesecake` - Agent features and inheritance (20+ tests)
- `test-control-flow.cheesecake` - Control flow constructs (25+ tests)
- `test-state.cheesecake` - State management and persistence (25+ tests)
- 90+ total test cases covering all language features

#### Documentation
- Complete language specification (2000+ lines)
- Quick syntax reference guide
- VM execution semantics documentation
- Helper agent methodology guide (1000+ lines)
- Comprehensive README with examples
- Inline code comments throughout

### Architecture
- AI-interpreted language (no traditional compiler)
- Session-as-runtime execution model
- Claude Code plugin architecture
- Subagent spawning via Task tool

---

## [0.0.2] - 2026-01-07 (In Progress)

### Added

#### Module 9: Progress & Dry-Run ✅ COMPLETE

**Progress Tracking**
- Real-time progress visualization with progress bars: `[■■■■■■□□□□] 60%`
- Three progress levels: statement-by-statement, phase-based, verbose mode
- Token usage tracking and budget warnings
- Time estimation and remaining time display
- Status indicators: ✓ (done), → (running), ○ (pending), ⚠ (warning), ✗ (failed)
- Progress update frequency: after each statement, every 2-3 seconds during long operations

**PHASE Construct (v0.0.2+)**
- Optional organizational blocks for better progress tracking
- Syntax: `PHASE "name": ... END PHASE`
- Sequential execution of phases
- Integration with checkpoints for resumability
- Phase-by-phase cost breakdown in estimates
- Rules: unique names, no nesting, works with all constructs
- Example: Research → Analysis → Writing → Output phases

**Cost Estimation**
- Pre-execution cost calculation using model-specific pricing
- Token estimation by operation type (simple vs complex tasks)
- Model pricing (2025 rates):
  - Sonnet 4.5: $3/$15 per million tokens
  - Opus 4.5: $15/$75 per million tokens
  - Haiku 3.5: $0.80/$4 per million tokens
- Complete estimation algorithm:
  1. Parse workflow and identify operations
  2. Calculate base costs per operation
  3. Account for multipliers (loops, parallel blocks)
  4. Add 20% buffer for context/retries
- Confidence levels: High (90%), Medium (60%), Low (40%)
- Cost estimation for all constructs: AGENT, SESSION, LOOP, PARALLEL, IF/ELSE
- Optimization suggestions based on expensive patterns

**Dry-Run Mode**
- Zero-cost workflow simulation without spawning agents
- Command: `/cheesecake run <file> --dry-run`
- 5-step execution protocol:
  1. Parse & validate syntax
  2. Build execution plan (count sessions, loops, phases)
  3. Simulate execution step-by-step
  4. Calculate costs using estimation formulas
  5. Display comprehensive summary
- Shows what WOULD happen without actual execution
- Catches errors before incurring costs
- Phase-by-phase simulation for complex workflows
- Comparison table: Normal Run vs Dry Run

**Verbose Mode**
- Command: `/cheesecake run <file> --dry-run --verbose`
- Detailed output: every statement execution, token breakdown, model selection reasoning
- Full cost calculations with intermediate steps
- Optimization opportunity identification

**Estimate Command**
- New command: `/cheesecake estimate <file>`
- Quick cost analysis without full simulation
- Faster than dry-run, focused on cost/time/token estimates
- Comparison mode: `/cheesecake estimate <file1> <file2>`
- Phase-by-phase cost breakdown
- Optimization suggestions:
  - Model selection recommendations
  - Parallel execution opportunities
  - Loop optimization strategies
  - Checkpoint placement suggestions
- Task complexity detection heuristics
- Loop iteration estimation (REPEAT, FOR, LOOP UNTIL)
- Parallel block time calculation (max time, not sum)
- Conditional branch cost averaging

**Documentation**
- `skills/cheesecake/progress.md` - 690 lines of progress tracking specification
- `skills/cheesecake/cost-estimation.md` - 800+ lines of cost calculation formulas
- `commands/cheesecake-run.md` - Updated with --dry-run and --verbose flags
- `commands/cheesecake-estimate.md` - 850+ lines for estimate command
- `skills/cheesecake/SKILL.md` - Updated with PHASE construct (170+ lines added)
- Total new documentation: 3,300+ lines

**Examples**
- Complete cost estimation examples (Hello World, Parallel Research, Iterative Loop)
- Phase-based workflow example (test-phase-workflow.cheesecake)
- Dry-run output examples with phase breakdown
- Comparison mode examples showing cost savings

### Testing
- Tested cost estimation on customer-feedback-analysis.cheesecake (10 sessions, $0.06)
- Tested phase-based estimation on test-phase-workflow.cheesecake (4 phases, $0.43)
- Validated progress tracking specifications
- Validated dry-run execution protocol

### Changed
- `SKILL.md` version updated to v0.0.2
- Added PHASE construct to Control Flow section (section 7.5)

### Backward Compatibility
- ✅ All v0.0.1 workflows continue to work
- PHASE blocks are optional
- Existing workflows without phases show statement-level progress
- No breaking changes to syntax

---

#### Module 10: Interactive Mode ✅ COMPLETE

**INTERACTIVE Construct**
- Human-in-the-loop workflows with pause/resume execution
- Syntax: `INTERACTIVE AT "checkpoint-name": SHOW: {var} ASK USER: "?" OPTIONS: ... END INTERACTIVE`
- Four components:
  - `AT "name"` - Unique identifier for interactive point
  - `SHOW: {variable}` - Display context before asking (optional)
  - `ASK USER: "question?"` - Question presented to user (required)
  - `OPTIONS:` - 2-10 multiple choice options with actions
- Zero cost during pause (no AI sessions spawned)
- State preservation: all variables, context, line number saved
- Seamless resume after user input

**Pause/Resume Execution**
- 5-step execution flow:
  1. Pre-pause state preservation
  2. Display context (if SHOW present)
  3. Present question and options (via AskUserQuestion tool)
  4. Execute selected action
  5. Resume execution
- Time and token counters pause during user input
- Progress tracking shows ⏸ indicator for paused phase
- Can resume from checkpoint if user disconnects

**Action Execution**
- Actions can be:
  - Variable assignments (`VAR x = true`)
  - Session executions (`RUN SESSION(...)`)
  - Control flow (`BREAK`, `CONTINUE`, `RETURN`)
  - Any valid CheeseCake statements
- User selection triggers immediate action execution
- Variables from actions available after INTERACTIVE block

**Integration with AskUserQuestion**
- INTERACTIVE uses Claude Code's `AskUserQuestion` tool internally
- Question from `ASK USER`
- Options from `OPTIONS` block with auto-generated descriptions
- Header from checkpoint name
- Single selection mode (not multi-select)

**Special Cases**
- ✅ INTERACTIVE in loops (iterative refinement with user feedback)
- ✅ INTERACTIVE in conditionals (conditional pause based on criteria)
- ✅ Sequential INTERACTIVE blocks (multi-step user input)
- ✅ Multiple INTERACTIVE blocks in same workflow
- ❌ INTERACTIVE inside PARALLEL blocks (parse error)
- ❌ Nested INTERACTIVE blocks (parse error)

**Progress Integration**
- Paused phase shows ⏸ symbol
- Clear pause indicator: `[PAUSE] Waiting for user input at: checkpoint-name`
- Time counter pauses during user input
- Token counter pauses (zero cost)
- Resumes seamlessly after input

**Use Cases**
- Approval workflows (cost approval, quality review, safety checks)
- Iterative refinement (user feedback at each iteration)
- Agent/model selection (let user choose Sonnet vs Opus)
- Path branching (user chooses workflow direction)
- Early exit points (user can cancel expensive operations)
- Quality checkpoints (pause if quality below threshold)

**Documentation**
- `skills/cheesecake/interactive.md` - 1,200+ lines of interactive mode specification
- `skills/cheesecake/SKILL.md` - Added INTERACTIVE construct (260+ lines, section 9)
- `skills/cheesecake/vm.md` - Added pause/resume semantics (350+ lines, section 8)
- Total new documentation: 1,810+ lines

**Examples**
- Simple approval workflow (proceed vs skip)
- Iterative draft review with loop
- Multi-agent/model selection
- Conditional INTERACTIVE (only if quality low)
- Cost-aware branching with sequential INTERACTIVE

**Test File**
- `test-interactive-workflow.cheesecake` - 180+ lines testing all features
- 5 comprehensive test scenarios
- All key features validated

### Testing (Module 10)
- Tested simple approval workflow ✅
- Tested iterative review in loop ✅
- Tested model selection (Sonnet/Opus/skip) ✅
- Tested conditional INTERACTIVE ✅
- Tested sequential INTERACTIVE blocks ✅
- Tested zero cost during pause ✅
- Validated all constraints (no PARALLEL, no nesting) ✅

### Changed (Module 10)
- `SKILL.md` sections renumbered (9-17)
- Added INTERACTIVE construct as section 9
- `vm.md` sections renumbered (8-16)
- Added Interactive Mode Execution as section 8

### Backward Compatibility (Module 10)
- ✅ All v0.0.1 and v0.0.2 workflows continue to work
- INTERACTIVE blocks are optional
- Existing workflows without INTERACTIVE blocks work perfectly
- No breaking changes to syntax

---

#### Module 11: Cost Management ✅ COMPLETE

**CONFIG Block**
- Global workflow configuration with cost management settings
- Syntax: `CONFIG: ... END CONFIG` (optional, at start of file)
- 11 configuration settings across 4 categories:
  - Cost Management: BUDGET, CONFIRM_COST_ABOVE, WARN_PARALLEL_ABOVE, WARN_AT_PERCENT, OPTIMIZATION_SUGGESTIONS
  - Model Settings: DEFAULT_MODEL
  - Execution Settings: MAX_PARALLEL_SESSIONS, TIMEOUT_DEFAULT
  - Behavior Settings: STOP_ON_BUDGET_EXCEED, INTERACTIVE_WARNINGS
- Must appear at start of file, only one per workflow
- All settings optional with sensible defaults

**Cost Tracking**
- Real-time cost tracking during workflow execution
- Track cost of each session based on model and token usage
- Accumulate running total throughout workflow
- Display cost in progress visualization
- Model pricing (2025 rates):
  - Sonnet 4.5: $3/$15 per million tokens
  - Opus 4.5: $15/$75 per million tokens
  - Haiku 3.5: $0.80/$4 per million tokens

**Warning System**
- Four types of cost warnings:
  1. Operation Cost Warning: Single operation exceeds threshold
  2. Parallel Session Warning: Many parallel sessions about to spawn
  3. Budget Threshold Warning: Budget usage reaches percentage
  4. Budget Exceeded Error: Budget limit exceeded
- Interactive warnings pause for user input
- Non-interactive mode logs warnings and auto-continues
- User options: Continue (Y), Skip (n), Edit task (e), Reduce count (r)

**Budget Enforcement**
- Hard budget limit: Stops workflow when budget exceeded
- Soft budget: Warns but continues execution
- Budget checks at key points:
  - Before each RUN SESSION
  - Before PARALLEL blocks
  - Before each LOOP iteration
  - At percentage thresholds
- Configurable via STOP_ON_BUDGET_EXCEED setting

**Optimization Suggestions**
- AI-powered suggestions for cost reduction
- Six types of suggestions:
  1. Model downgrade (Opus → Sonnet)
  2. Parallelize sequential operations
  3. Add checkpoints to prevent re-runs
  4. Batch instead of loop
  5. Cache repeated operations
  6. Use CHOICE ON to skip unnecessary branches
- Shown after workflow completion
- Displays potential savings and quality impact
- Can be disabled via OPTIMIZATION_SUGGESTIONS: false

**Progress Integration**
- Cost displayed in progress visualization:
  ```
  Cost: $0.12 / $1.00 budget (12% used) ✓
  ```
- Three indicators:
  - ✓ Within budget (<75%)
  - ⚠ Approaching budget (75-100%)
  - ❌ Budget exceeded (>100%)
- Real-time updates after each session
- Estimated cost shown during execution

**Documentation**
- `skills/cheesecake/cost-management.md` - 1,050+ lines of specification
- `skills/cheesecake/SKILL.md` - Added CONFIG block (section 11, 280+ lines)
- `skills/cheesecake/vm.md` - Added cost tracking semantics (section 9, 400+ lines)
- `test-cost-management.cheesecake` - 330 lines testing all features
- Total new documentation: 2,060+ lines

**Examples**
- Basic budget control
- Production configuration (conservative)
- Development configuration (permissive)
- Parallel session warning
- Budget exceeded scenario
- Multi-phase with optimization

### Testing (Module 11)
- Tested CONFIG block parsing ✅
- Tested cost tracking simulation ✅
- Tested warning triggers ✅
- Tested budget enforcement logic ✅
- Tested optimization suggestion generation ✅
- Validated progress integration ✅
- Validated 15 comprehensive test scenarios ✅

### Changed (Module 11)
- `SKILL.md` sections renumbered (12-18, was 11-17)
- `vm.md` sections renumbered (10-17, was 9-16)
- Added CONFIG block as section 11 in SKILL.md
- Added Cost Management as section 9 in vm.md

### Backward Compatibility (Module 11)
- ✅ All v0.0.1 and v0.0.2 workflows continue to work
- CONFIG block is completely optional
- Default: no budget limit, no warnings, no restrictions
- Existing workflows without CONFIG run normally
- No breaking changes to syntax

---

## [Unreleased] - Planned for future v0.0.2 modules

### Planned Features (Modules 12-14)
- Module 12: Event handlers and scheduling (ON EVENT, SCHEDULE)
- Module 13: Enhanced testing features (TEST SUITE, MOCK, ASSERT)
- Module 14: Execution history and replay

---

## Contributors

- Rupesh Raj - Creator and maintainer

---

**For detailed documentation, see [README.md](README.md)**
