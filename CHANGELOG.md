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

## [Unreleased] - Planned for future v0.0.2 modules

### Planned Features (Modules 10-14)
- Module 10: Interactive mode with user input prompts (INTERACTIVE construct)
- Module 11: Cost management with CONFIG block and budget enforcement
- Module 12: Event handlers and scheduling (ON EVENT, SCHEDULE)
- Module 13: Enhanced testing features (TEST SUITE, MOCK, ASSERT)
- Module 14: Execution history and replay

---

## Contributors

- Rupesh Raj - Creator and maintainer

---

**For detailed documentation, see [README.md](README.md)**
