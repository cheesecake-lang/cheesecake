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

#### Module 12: Events & Scheduling ✅ COMPLETE

**ON EVENT Construct**
- Event handlers that respond to triggers
- Syntax: `ON EVENT name(params) [WHERE condition]: ... END ON`
- Built-in event types (declarative):
  - `file_changed(path, type)` - File system changes
  - `file_created(path)` - New file created
  - `file_deleted(path)` - File deleted
  - `api_webhook(endpoint, payload)` - HTTP webhook
  - `timer_tick(timestamp)` - Timer events
  - `user_input(data)` - User-triggered input
  - `session_start(session_id)` - Session started
  - `session_end(session_id, result)` - Session completed
- Custom events via EMIT
- WHERE clause filtering:
  - Literal comparisons (`status == 200`)
  - Pattern matching (`path MATCHES "*.ts"`)
  - Semantic conditions (`**{payload} contains error**`)
  - Combined conditions with AND/OR
- Multiple handlers per event (execute in order)
- Handler error isolation (errors don't stop other handlers)

**SCHEDULE Construct**
- Time-based scheduled tasks
- Syntax: `SCHEDULE name: ... END SCHEDULE`
- Three timing types:
  - `INTERVAL: Nh` - Fixed intervals (Ns, Nm, Nh, Nd, Nw)
  - `CRON: "expression"` - Cron expressions (minute hour day month weekday)
  - `ONCE_AT: "timestamp"` - Single execution at specific time
- Schedule properties:
  - `START_AT` - When schedule becomes active
  - `END_AT` - When schedule deactivates
  - `TASK` - Statement or block to execute
  - `RETRY` - Retry count on failure
  - `ON_FAILURE` - Action when task fails
  - `ON_SUCCESS` - Action when task succeeds
- Manual trigger: `/cheesecake trigger <schedule_name>`

**EMIT Construct**
- Trigger custom events
- Syntax: `EMIT event_name(param: value, ...)`
- Named parameters with values
- Dispatched immediately to matching handlers
- Event chain depth limit (MAX_EVENT_DEPTH: 10)

**LISTEN FOR Construct**
- Lightweight internal event listener
- Syntax: `LISTEN FOR event_name: ... END LISTEN`
- Only handles internal events (from EMIT)
- Access event data via `event.*` notation
- Simpler than ON EVENT (no WHERE clause)

**Documentation**
- `skills/cheesecake/events.md` - 870+ lines of specification
- `skills/cheesecake/SKILL.md` - Added Events & Scheduling (section 12, 160+ lines)
- `skills/cheesecake/vm.md` - Added Event Execution (section 10, 340+ lines)
- `test-events.cheesecake` - 340 lines testing all features
- Total new documentation: 1,710+ lines

**Examples**
- File change handlers with filtering
- Scheduled health checks
- Daily report generation (CRON)
- One-time scheduled deployment
- Event chains and coordination
- Error handling in handlers

### Testing (Module 12)
- Tested ON EVENT declaration ✅
- Tested WHERE clause filtering ✅
- Tested EMIT custom events ✅
- Tested LISTEN FOR internal events ✅
- Tested SCHEDULE (INTERVAL, CRON, ONCE_AT) ✅
- Tested event chains ✅
- Tested error isolation ✅
- Tested complex event data ✅
- Tested MATCHES pattern filtering ✅
- Validated 15 test scenarios ✅

### Changed (Module 12)
- `SKILL.md` sections renumbered (13-19, was 12-18)
- `vm.md` sections renumbered (11-18, was 10-17)
- Added Events & Scheduling as section 12 in SKILL.md
- Added Event Execution as section 10 in vm.md

### Backward Compatibility (Module 12)
- ✅ All v0.0.1 and v0.0.2 workflows continue to work
- Events and schedules are optional
- Existing workflows without events work perfectly
- No breaking changes to syntax

**Note on Execution:**
- External events are declarative (require runtime integration)
- Internal events (EMIT/LISTEN) work fully within sessions
- Schedules can be manually triggered for testing
- Future: CheeseCake daemon for real event monitoring

---

#### Module 13: Testing Framework ✅ COMPLETE

**TEST SUITE Construct**
- Groups related tests with shared setup/teardown
- Syntax: `TEST SUITE "name": ... END TEST SUITE`
- SETUP block runs before each test
- TEARDOWN block runs after each test (even on failure)
- Multiple tests per suite, execution in order

**TEST Construct**
- Individual test cases
- Syntax: `TEST "name": ... END TEST`
- Isolated scope per test
- First failed ASSERT stops test
- Can be standalone or inside suite

**MOCK Construct**
- Replace agent sessions with fixed responses
- Syntax variations:
  - `MOCK Agent RETURNS value` - Simple mock
  - `MOCK Agent FOR TASK MATCHING "pattern" RETURNS value` - Task-specific
  - `MOCK Agent THROWS "error"` - Error simulation
- Pattern matching: `*`, `*keyword*`, `prefix *`, `* suffix`
- Mock scope limited to current TEST block
- Zero cost (no AI calls)

**ASSERT Construct**
- Verify conditions in tests
- Literal assertions:
  - `==`, `!=`, `>`, `<`, `>=`, `<=`
  - `IS NULL`, `IS NOT NULL`
  - `IS TRUE`, `IS FALSE`
  - `CONTAINS`, `MATCHES`
- Semantic assertions: `ASSERT **{value} is quality check**`
- Custom messages: `ASSERT condition MESSAGE "text"`

**/cheesecake test Command**
- Run tests: `/cheesecake test [file] [options]`
- Options: `--suite`, `--test`, `--verbose`, `--fail-fast`, `--strict`
- Output formats: text (default), json, junit
- Test discovery and execution
- Pass/fail reporting with details

**Documentation**
- `skills/cheesecake/testing.md` - 750+ lines of specification
- `skills/cheesecake/SKILL.md` - Added Testing Framework (section 13, 155+ lines)
- `skills/cheesecake/vm.md` - Added Test Execution (section 11, 270+ lines)
- `commands/cheesecake-test.md` - 200+ lines for test command
- `test-testing-framework.cheesecake` - 400+ lines testing all features
- Total new documentation: 1,775+ lines

**Examples**
- Basic test suites with setup/teardown
- Simple and task-specific mocks
- Error simulation with MOCK THROWS
- All assertion operators
- Semantic quality assertions
- Test isolation verification
- Integration test patterns

### Testing (Module 13)
- Tested TEST SUITE declaration ✅
- Tested SETUP/TEARDOWN ✅
- Tested simple MOCK ✅
- Tested task-specific MOCK patterns ✅
- Tested MOCK THROWS ✅
- Tested literal ASSERT operators ✅
- Tested semantic ASSERT ✅
- Tested custom ASSERT MESSAGE ✅
- Tested multiple mocks ✅
- Tested test isolation ✅
- Tested standalone TEST ✅
- Tested nested data assertions ✅
- Tested error handling in tests ✅
- Tested loops in tests ✅
- Validated 15 test scenarios ✅

### Changed (Module 13)
- `SKILL.md` sections renumbered (14-20, was 13-19)
- `vm.md` sections renumbered (12-19, was 11-18)
- Added Testing Framework as section 13 in SKILL.md
- Added Test Execution as section 11 in vm.md
- Added /cheesecake test command

### Backward Compatibility (Module 13)
- ✅ All v0.0.1 and v0.0.2 workflows continue to work
- Testing constructs are optional
- Existing workflows without tests work perfectly
- No breaking changes to syntax

---

#### Program Contracts & Composition ✅ COMPLETE

**INPUT Declaration**
- Declare expected inputs for reusable programs
- Syntax: `INPUT name: "description"` (required)
- Syntax: `INPUT name: "description" DEFAULT: value` (optional)
- Supports all types: strings, numbers, booleans, arrays, objects
- INPUTs become variables available in program scope
- Contract validation at call time:
  - Missing required input → ERROR
  - Unknown input → WARNING (ignored)
  - Missing optional → Uses DEFAULT value

**OUTPUT Declaration**
- Declare returned values from programs
- Syntax: `OUTPUT name = expression`
- Multiple outputs supported
- Outputs collected into result object
- Caller accesses via dot notation: `result.name`

**USE Statement**
- Import programs for composition
- Local path: `USE "./path/to/program.cheesecake" AS alias`
- Registry: `USE "@handle/slug"` or `USE "@namespace/program-name" AS alias`
- Creates alias in current scope
- Can USE multiple programs
- Path resolution: relative to current file (local) or from registry (remote)

**Registry URL Resolution**
- Registry path format: `@<handle>/<slug>`
- Resolution URL: `https://registry.cheesecake.dev/@handle/slug`
- Resolution steps:
  1. Parse @handle/slug
  2. Check local cache first
  3. Fetch from registry if not cached/expired
  4. Parse and validate program
  5. Register in ImportRegistry

**Registry Caching**
- Cache location: `.cheesecake/cache/@handle/slug`
- Cache metadata: timestamp, version, etag
- Default TTL: 24 hours
- Falls back to cache on network errors
- Force refresh: `USE "@workflows/web-researcher" REFRESH`

**Destructuring Outputs**
- Standard: `VAR result = RUN program(...)`
- Destructured: `VAR { findings, sources } = RUN program(...)`
- With rename: `VAR { findings AS data } = RUN program(...)`
- Partial: `VAR { findings } = RUN program(...)`

**Import Registry**
- VM maintains ImportRegistry tracking all USE'd programs
- Stores: source, source_type (LOCAL/REGISTRY), contract, cached_at, version
- Helper functions: `HAS_IMPORT("name")`, `GET_INPUTS("name")`, `GET_OUTPUTS("name")`

**Program Invocation**
- Call imported programs with RUN
- Syntax: `VAR result = RUN program_name(input1: value1, input2: value2)`
- Named arguments required
- Returns object with OUTPUT properties
- Access outputs: `result.findings`, `result.metadata.generated`

**Parallel Program Execution**
- Programs can run in parallel
- Syntax: `PARALLEL: VAR r1 = RUN prog(a: 1) VAR r2 = RUN prog(a: 2) END PARALLEL`
- All results available after block completes

**Program Composition Patterns**
- Chain programs: output of one → input of another
- Nested invocation with depth tracking
- MAX_PROGRAM_DEPTH: 10 (prevents infinite recursion)
- Error handling with TRY/CATCH

**IMPORT vs USE**
- IMPORT: Include code definitions (agents, skills, functions)
- USE: Call complete programs with contracts
- Both can be used together in same workflow

**Documentation**
- `skills/cheesecake/SKILL.md` - Added Section 3: Program Contracts (160+ lines)
- `skills/cheesecake/SKILL.md` - Updated Section 17: Modules, Imports & Composition (350+ lines)
- `skills/cheesecake/vm.md` - Added Section 12: Program Contracts & Composition (450+ lines)
- `test-program-contracts.cheesecake` - 610 lines testing all features
- Total new documentation: 1,570+ lines

**Test Coverage (24 tests)**
- TEST 1: INPUT declaration (required)
- TEST 2: INPUT declaration (optional with DEFAULT)
- TEST 3: OUTPUT declaration
- TEST 4: Multiple INPUT types
- TEST 5: USE statement (local path)
- TEST 6: USE statement (registry path)
- TEST 7: Program invocation syntax
- TEST 8: Accessing program outputs
- TEST 9: Parallel program execution
- TEST 10: Program composition chain
- TEST 11: Error handling in program calls
- TEST 12: Contract validation
- TEST 13: INPUT as variables
- TEST 14: Complete program example
- TEST 15: USE vs IMPORT comparison
- TEST 16: Registry URL resolution
- TEST 17: Destructuring outputs
- TEST 18: Registry caching
- TEST 19: Force refresh
- TEST 20: Import registry structure
- TEST 21: Complete registry composition
- TEST 22: Invoking with destructuring
- TEST 23: Registry error handling
- TEST 24: Registry helper functions

### Testing (Program Contracts)
- Tested INPUT declaration (required/optional) ✅
- Tested OUTPUT binding ✅
- Tested USE statement syntax (local & registry) ✅
- Tested program invocation ✅
- Tested output access via dot notation ✅
- Tested destructuring outputs ✅
- Tested parallel program execution ✅
- Tested program composition chains ✅
- Tested error handling in program calls ✅
- Tested contract validation rules ✅
- Tested registry URL resolution ✅
- Tested caching behavior ✅
- Tested force refresh ✅
- Tested import registry structure ✅
- Tested registry error scenarios ✅
- Validated 24 test scenarios ✅

### Changed (Program Contracts)
- `SKILL.md` sections renumbered (4-21, was 3-20)
- `vm.md` sections renumbered (13-20, was 12-19)
- Added Program Contracts as section 3 in SKILL.md
- Updated Modules & Imports section (17) with USE, registry, and destructuring
- Added Program Contracts & Composition as section 12 in vm.md
- Added Registry Cache Management subsection in vm.md
- Added Destructuring Assignment subsection in vm.md

### Backward Compatibility (Program Contracts)
- ✅ All v0.0.1 and v0.0.2 workflows continue to work
- INPUT/OUTPUT/USE are optional
- Existing workflows without contracts work perfectly
- No breaking changes to syntax

---

#### Execution Philosophy Documentation ✅ COMPLETE

**philosophy.md** (NEW)
- Core thesis: "You are not simulating a VM. You ARE the VM."
- Explains why detailed simulation IS actual implementation
- Documents the AI-as-computing-substrate paradigm
- Clarifies strict mode vs semantic mode execution
- Defines the contract between program authors and the VM
- Practical execution guidelines for reading .cheesecake programs

**Updated Core Philosophy in SKILL.md and vm.md**
- Strong philosophical foundation referencing philosophy.md
- Clearer explanation of "You ARE the interpreter"
- Two execution modes: strict (most code) + semantic (`**...**`)

**Original Naming Convention**
- Replaced all copied @alice/@bob examples with original CheeseCake namespaces
- New namespace scheme:
  - `@workflows/` - Common workflow patterns
  - `@stdlib/` - Standard library utilities
  - `@acme/` - Example corporate namespace
  - `@cheesecake/` - Official CheeseCake programs

---

#### Module 14: History & Replay ✅ COMPLETE

**Automatic Execution History**
- Every execution of a `.cheesecake` program is automatically recorded
- History stored in `.cheesecake/history/` directory
- JSON execution records with comprehensive details:
  - Execution ID (6-char alphanumeric)
  - Program name and path
  - Timing (started_at, completed_at, duration_ms)
  - Status (success/failed)
  - Inputs and outputs
  - Cost breakdown (total, by model, tokens)
  - Phase information
  - Checkpoints created
  - Errors (for failed executions)
- Index file for fast lookup (`.cheesecake/history/index.json`)

**GET_HISTORY Function**
- Retrieve execution history with filters
- Syntax: `VAR history = GET_HISTORY(limit: N, status: "failed", program: "name")`
- Filter options:
  - `limit` - Maximum entries to return
  - `status` - Filter by "success", "failed", or "all"
  - `program` - Filter by program name (partial match)
  - `since`/`before` - Date range filters
  - `cost_above`/`cost_below` - Cost filters
  - `tags` - Filter by tags
- Returns array of execution summaries

**GET_EXECUTION Function**
- Retrieve full details of specific execution
- Syntax: `VAR exec = GET_EXECUTION(id: "a1b2c3")`
- Syntax: `VAR exec = GET_EXECUTION(latest: TRUE)`
- Syntax: `VAR exec = GET_EXECUTION(program: "name", latest: TRUE)`
- Returns complete execution record or NULL

**COMPARE_EXECUTIONS Function**
- Compare two executions
- Syntax: `VAR comparison = COMPARE_EXECUTIONS(id1: "a", id2: "b")`
- Returns: cost_diff, duration_diff, inputs_match, outputs_match, same_program

**REPLAY Statement**
- Replay previous executions
- Basic: `REPLAY execution_id: "a1b2c3"`
- With modified inputs: `REPLAY execution_id: "a1b2c3" WITH new_inputs`
- From checkpoint: `REPLAY execution_id: "a1b2c3" FROM_CHECKPOINT: "checkpoint-name"`
- Creates new history entry with parent reference

**CLEAR_HISTORY Function**
- Clear execution history
- Syntax: `CLEAR_HISTORY()`
- With filters: `CLEAR_HISTORY(before: "date", status: "failed", program: "name")`
- Returns count of deleted entries

**History Configuration**
- 8 CONFIG settings for history behavior:
  - `HISTORY_ENABLED` - Enable/disable history (default: TRUE)
  - `HISTORY_RETENTION_DAYS` - Days to keep history (default: 30)
  - `HISTORY_MAX_ENTRIES` - Maximum entries to keep (default: 100)
  - `HISTORY_INCLUDE_OUTPUTS` - Store outputs (default: TRUE)
  - `HISTORY_OUTPUT_MAX_SIZE` - Truncate large outputs (default: 10000)
  - `HISTORY_INCLUDE_INPUTS` - Store inputs (default: TRUE)
  - `HISTORY_REDACT_SECRETS` - Redact sensitive values (default: TRUE)
  - `HISTORY_TAGS` - Default tags for executions

**History Events**
- `execution_recorded(record)` - Fired after execution recorded
- `history_cleanup(deleted_count)` - Fired after cleanup

**Commands**
- `/cheesecake history` - List recent executions
- `/cheesecake history #N` - Show execution details
- `/cheesecake history --status failed` - Filter by status
- `/cheesecake history --program name` - Filter by program
- `/cheesecake history --since date` - Filter by date
- `/cheesecake history --cost-above N` - Filter by cost
- `/cheesecake history --stats` - Show statistics
- `/cheesecake history --compare #A #B` - Compare executions
- `/cheesecake history --clear` - Clear history
- `/cheesecake replay #N` - Replay execution
- `/cheesecake replay #N --modify` - Replay with modified inputs
- `/cheesecake replay #N --from checkpoint` - Resume from checkpoint

**Documentation**
- `skills/cheesecake/history.md` - 670+ lines of specification
- `skills/cheesecake/SKILL.md` - Added History & Replay (section 22, 320+ lines)
- `skills/cheesecake/vm.md` - Added History Tracking (section 18, 380+ lines)
- `commands/cheesecake-history.md` - 550+ lines for history command
- `commands/cheesecake-replay.md` - 350+ lines for replay command
- `tests/test-history.cheesecake` - 450+ lines testing all features
- `MODULE-14-PLAN.md` - Detailed implementation plan
- Total new documentation: 2,720+ lines

**Test Coverage (45+ tests in 12 suites)**
- GET_HISTORY function tests (7 tests)
- GET_EXECUTION function tests (7 tests)
- COMPARE_EXECUTIONS function tests (3 tests)
- REPLAY statement tests (3 tests)
- CLEAR_HISTORY function tests (4 tests)
- History configuration tests (6 tests)
- History events tests (2 tests)
- Failed execution history tests (3 tests)
- Cost tracking in history tests (3 tests)
- Replay metadata tests (3 tests)
- History index tests (3 tests)
- Integration test (1 test)

### Testing (Module 14)
- Tested GET_HISTORY with all filter options ✅
- Tested GET_EXECUTION by ID and latest ✅
- Tested COMPARE_EXECUTIONS ✅
- Tested REPLAY with same inputs ✅
- Tested REPLAY with modified inputs ✅
- Tested REPLAY from checkpoint ✅
- Tested CLEAR_HISTORY with filters ✅
- Tested history configuration options ✅
- Tested secret redaction ✅
- Tested output truncation ✅
- Tested failed execution recording ✅
- Tested cost tracking in history ✅
- Tested replay metadata ✅
- Validated 45+ test scenarios ✅

### Changed (Module 14)
- `SKILL.md` sections renumbered (added section 22: History & Replay)
- `vm.md` sections renumbered (19-21, was 18-20, added section 18: History Tracking)
- Added History & Replay as section 22 in SKILL.md
- Added History Tracking as section 18 in vm.md
- Added /cheesecake history command
- Added /cheesecake replay command

### Backward Compatibility (Module 14)
- ✅ All v0.0.1 and v0.0.2 workflows continue to work
- History tracking is automatic (can be disabled via CONFIG)
- History functions are optional
- Existing workflows work perfectly without changes
- No breaking changes to syntax

---

## [Unreleased] - Future planned features

### Planned for v0.0.3
- Visual workflow builder
- IDE extension (VSCode)
- Plugin marketplace

---

## Contributors

- Rupesh Raj - Creator and maintainer

---

**For detailed documentation, see [README.md](README.md)**
