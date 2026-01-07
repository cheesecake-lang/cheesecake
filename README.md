# CheeseCake

> **A structured OOP programming language for AI agent orchestration**

CheeseCake is a programming language designed for orchestrating AI agents. It treats long-running AI sessions as Turing-complete computers and provides a structured yet natural syntax for building complex agentic workflows.

---

## Key Features

- **🎯 Zero Learning Curve**: Helper agent generates `.cheesecake` files from natural language descriptions
- **🏗️ Object-Oriented**: First-class `AGENT` (classes), `SKILL` (traits), and `FUNCTION` definitions
- **🔄 Resumable**: Built-in checkpoints for long-running workflows
- **🌐 Portable**: Works as an abstraction layer across AI agents (not vendor-locked)
- **🧪 Testable**: Built-in testing framework with mocks and assertions
- **📦 Composable**: Full module system with `IMPORT`/`EXPORT`

---

## Quick Start

### Installation (Claude Code)

**Option 1: Install from Claude Code Plugin Marketplace** (Recommended)

```bash
# Install CheeseCake from the official plugin marketplace
# (In Claude Code CLI)
/plugin install cheesecake@claude-plugins-official

# Verify installation
/cheesecake
```

**Option 2: Install from Source** (For development)

```bash
# Clone the repository
git clone https://github.com/cheesecake-lang/cheesecake.git
cd cheesecake

# Install as Claude Code plugin from local directory
# (In Claude Code CLI, from within the cheesecake directory)
/plugin install ./

# Verify installation
/cheesecake
```

### Your First CheeseCake Program

Create `hello.cheesecake`:

```cheesecake
# ============================================
# Hello CheeseCake!
# A simple program to demonstrate the basics
# ============================================

# Define an agent (like a class)
AGENT Greeter:
  MODEL: sonnet
  PROMPT: "You are a friendly greeter."

# Create an instance
VAR greeter = NEW Greeter()

# Run a task
VAR message = RUN SESSION(greeter):
  TASK: "Say hello to the world of AI orchestration!"

# Output the result
PRINT message
```

Run it:
```bash
/cheesecake run hello.cheesecake
```

---

## Language Overview

### Core Concepts

| Concept | Description | Example |
|---------|-------------|---------|
| **SKILL** | Reusable capabilities (traits) | `SKILL web-research:` |
| **AGENT** | Agent templates (classes) | `AGENT Researcher:` |
| **SESSION** | Agent instance | `SESSION(agent): TASK: "..."` |
| **RUN** | Execute session | `RUN SESSION(...)` |
| **VAR/CONST** | Variables | `VAR x = ...` |
| **FUNCTION** | Reusable workflows | `FUNCTION name(args): ... RETURN` |

### Syntax Highlights

```cheesecake
# 1. Define capabilities (traits/interfaces)
SKILL research-capabilities:
  CAPABILITIES: [web-search, data-analysis]

# 2. Define agent classes
AGENT Researcher:
  MODEL: sonnet
  SKILLS: [research-capabilities]
  PROMPT: "You are a thorough researcher."

# 3. Inheritance
AGENT SeniorResearcher EXTENDS Researcher:
  MODEL: opus  # Override
  SKILLS: [+strategic-thinking]  # Add

# 4. Create instances and run tasks
VAR researcher = NEW Researcher()
VAR findings = RUN SESSION(researcher):
  TASK: "Research quantum computing trends"

# 5. Control flow
IF **{findings} shows significant progress**:
  PRINT "Exciting developments!"
END IF

# 6. Loops with semantic conditions
LOOP UNTIL **{analysis} is complete** MAX 5:
  VAR analysis = RUN SESSION(analyst): TASK: "Analyze {findings}"
END LOOP

# 7. Parallel execution
PARALLEL:
  VAR research1 = RUN SESSION(r1): TASK: "Research topic A"
  VAR research2 = RUN SESSION(r2): TASK: "Research topic B"
END PARALLEL

# 8. State persistence
CHECKPOINT "research-phase":
  SAVE: {findings, analysis}
END CHECKPOINT

# 9. Functions
FUNCTION analyze_topic(topic):
  VAR data = RUN SESSION(researcher): TASK: "Research {topic}"
  RETURN data
END FUNCTION
```

---

## Commands

| Command | Description |
|---------|-------------|
| `/cheesecake` | Main menu (detect files, show options) |
| `/cheesecake run <file>` | Execute a .cheesecake file |
| `/cheesecake create` | Helper wizard to create new file |
| `/cheesecake validate <file>` | Validate syntax |
| `/cheesecake explain <file>` | Explain what a file does |
| `/cheesecake help [topic]` | Get help |
| `/cheesecake init` | Initialize CheeseCake in project |

---

## Example: Research & Write Workflow

```cheesecake
# research-article.cheesecake

# Define agents
AGENT Researcher:
  MODEL: sonnet
  SKILLS: [web-research]

AGENT Writer:
  MODEL: opus
  SKILLS: [content-creation]

# Create instances
VAR researcher = NEW Researcher()
VAR writer = NEW Writer()

# Parallel research
PARALLEL:
  VAR academic = RUN SESSION(researcher):
    TASK: "Find academic papers on quantum computing"
  VAR industry = RUN SESSION(researcher):
    TASK: "Analyze industry developments"
END PARALLEL

# Checkpoint for resumability
CHECKPOINT "research-complete":
  SAVE: {academic, industry}
END CHECKPOINT

# Iterative writing
VAR draft = RUN SESSION(writer):
  TASK: "Write article using research"
  INPUT: {academic, industry}

LOOP UNTIL **{draft} is publication-ready** MAX 5:
  VAR feedback = RUN SESSION(writer): TASK: "Review and improve {draft}"
  VAR draft = RUN SESSION(writer): TASK: "Refine based on {feedback}"
END LOOP

# Output
SAVE draft TO "output/article.md"
LOG SUCCESS: "Article complete!"
```

---

## Why CheeseCake vs Other Tools?

### vs OpenProse

| Feature | OpenProse | CheeseCake |
|---------|-----------|------------|
| Paradigm | Procedural | Object-Oriented |
| Agent Inheritance | ❌ | ✅ `EXTENDS` |
| Helper System | ❌ | ✅ Zero-learning-curve |
| State Persistence | Fragile (emoji) | Robust (checkpoints) |
| Testing | ❌ | ✅ Built-in |
| Functions | Limited blocks | Full `FUNCTION`/`RETURN` |

### vs LangChain/CrewAI

| Feature | LangChain | CheeseCake |
|---------|-----------|------------|
| Execution | External orchestration | AI-native (session-as-runtime) |
| Language | Python code | Natural structured syntax |
| Portability | Python-locked | Works across AI agents |
| Resumability | Manual | Built-in checkpoints |

---

## Architecture

### How It Works

```
┌─────────────────────────────────────────┐
│  User writes .cheesecake file          │
│  (or uses Helper to generate it)       │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│  CheeseCake VM (AI Agent)              │
│  - Reads the .cheesecake file          │
│  - Understands syntax via SKILL        │
│  - Executes according to VM semantics  │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│  Creates Task (sub-agent sessions)     │
│  - Spawns agents as defined            │
│  - Coordinates parallel execution      │
│  - Manages state/checkpoints           │
└──────────────┬──────────────────────────┘
               │
               ▼
        ┌──────────────┐
        │  Your Output │
        └──────────────┘
```

### No Traditional Compiler

CheeseCake is **AI-interpreted**, not compiled:
- The AI reads `.cheesecake` files directly
- Syntax is understood through language specification (Skills)
- Execution happens through intelligent interpretation
- This is **structured prompt engineering**, not traditional programming

---

## Project Structure

```
cheesecake/
├── .claude-plugin/
│   └── plugin.json              # Plugin manifest
├── skills/
│   └── cheesecake/
│       ├── SKILL.md             # Core language spec
│       ├── vm.md                # VM execution semantics
│       ├── helper.md            # Helper agent instructions
│       └── syntax-reference.md  # Quick reference
├── agents/
│   ├── vm/
│   │   └── AGENT.md            # CheeseCake VM agent
│   └── helper/
│       └── AGENT.md            # Helper agent
├── commands/
│   ├── cheesecake.md           # Main entry
│   ├── cheesecake-run.md       # Execute files
│   ├── cheesecake-create.md    # Helper wizard
│   └── ...
├── templates/
│   ├── hello-world.cheesecake
│   ├── research-and-write.cheesecake
│   └── ...
├── tests/
│   └── ...
└── README.md                    # This file
```

---

## Documentation

- **[Language Reference](skills/cheesecake/syntax-reference.md)** - Complete syntax guide
- **[VM Semantics](skills/cheesecake/vm.md)** - How execution works
- **[Examples](templates/)** - Pre-built templates
- **[Helper Guide](skills/cheesecake/helper.md)** - Using the Helper agent

---

## Roadmap

| Version | Focus | Status |
|---------|-------|--------|
| **v0.0.1** | Core MVP | 🚧 In Progress |
| **v0.0.2** | UX Enhancement (Dry-run, Interactive, Cost estimation) | 📅 Planned |
| **v0.0.3** | Ecosystem (IDE extension, Visual builder) | 📅 Planned |

---

## Contributing

CheeseCake is under active development. Contributions welcome!

### Development Setup

1. Clone the repo
2. Load as Claude Code plugin
3. Test with example files
4. Submit PRs

---

## License

MIT License - see LICENSE file

---

## Credits

**Author**: Rupesh Raj

Inspired by:
- **OpenProse** - Session-as-runtime concept
- **Claude Code** - AI-native development environment
- **Turing's Vision** - AI agents as universal computers

---

**Built with ❤️ for the agentic AI future**
