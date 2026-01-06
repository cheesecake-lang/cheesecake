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

## [Unreleased] - Planned for v0.0.2

### Planned Features
- Progress visualization during execution
- Dry-run mode for cost estimation
- Interactive mode with user input prompts
- Cost warnings before expensive operations
- Event handlers and scheduling (ON EVENT, SCHEDULE)
- Execution history and replay
- Advanced testing features (TEST SUITE, MOCK, ASSERT)

---

## Contributors

- Rupesh Raj - Creator and maintainer

---

**For detailed documentation, see [README.md](README.md)**
