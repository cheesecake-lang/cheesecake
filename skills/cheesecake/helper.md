# CheeseCake Helper Skill
# Purpose: Convert natural language workflow descriptions into valid .cheesecake files
# Part of: CheeseCake v0.0.1 - Module 5: Helper Agent
#
# This skill enables the Helper agent to understand natural language descriptions
# and generate working CheeseCake code, providing a zero-learning-curve entry point.

---

## Overview

The CheeseCake Helper is an AI agent that:
1. **Listens** to natural language descriptions of workflows
2. **Analyzes** the intent and breaks down into algorithmic steps
3. **Generates** valid `.cheesecake` code implementing the workflow
4. **Explains** the generated code to the user
5. **Refines** based on user feedback

---

## Core Competencies

### 1. Intent Understanding

When a user describes a workflow, identify:

- **Goal**: What is the end result? (article, analysis, decision, etc.)
- **Inputs**: What data/information is needed?
- **Agents needed**: What types of AI agents are required? (researcher, writer, analyst, etc.)
- **Flow type**: Is it sequential, parallel, iterative, or conditional?
- **Constraints**: Time limits, cost limits, quality requirements
- **Output format**: How should results be saved/displayed?

**Examples:**

```
User: "I want to research a topic and write an article about it"

Analysis:
- Goal: Article (text document)
- Inputs: Topic (user will provide)
- Agents: Researcher (gather info), Writer (create content)
- Flow: Sequential (research → write)
- Output: Save to file
```

```
User: "Monitor my codebase and run tests when files change"

Analysis:
- Goal: Automated testing
- Inputs: File system events
- Agents: Linter, Tester
- Flow: Event-driven (on file change)
- Constraints: Must be reactive
- Output: Test results, fix suggestions
```

---

### 2. Algorithmic Decomposition

Break natural language into clear algorithmic steps:

**Pattern Recognition:**

| User Says | Algorithm Pattern |
|-----------|-------------------|
| "research X and write about it" | Sequential: Research → Write |
| "compare A and B" | Parallel: Research(A) ‖ Research(B) → Compare |
| "keep improving until good" | Loop with semantic condition |
| "if X then Y, otherwise Z" | Conditional branching |
| "review and fix code" | Sequential with potential iteration |
| "process each item in list" | For-each loop |
| "run every hour" | Scheduled task |
| "when file changes" | Event handler |

**Step Granularity:**
- Each step should be a single, clear action an AI agent can perform
- Steps should have clear inputs and outputs
- Steps should be ordered logically

**Example Decomposition:**

```
User: "Research quantum computing, analyze the findings, and write a technical article.
       Keep refining the article until it's publication-ready."

Algorithm:
1. CREATE Researcher agent with web-search skills
2. CREATE Analyst agent with data-analysis skills
3. CREATE Writer agent with content-creation skills
4. RUN research task → findings
5. RUN analysis task with findings → analysis
6. RUN write task with analysis → draft
7. LOOP:
   a. RUN review task with draft → feedback
   b. IF feedback indicates ready → EXIT LOOP
   c. RUN revision task with draft + feedback → draft
   d. MAX 5 iterations
8. SAVE draft to file
```

---

### 3. CheeseCake Code Generation

Map algorithmic steps to CheeseCake syntax:

#### Agent Selection Guidelines

| Task Type | Recommended Agent | Model | Skills |
|-----------|------------------|-------|--------|
| Web research | Researcher | sonnet | web-research |
| Data analysis | Analyst | sonnet | data-analysis |
| Writing/content | Writer | opus | content-creation |
| Code review | CodeReviewer | sonnet | code-analysis |
| Editing | Editor | sonnet | editing |
| Strategic planning | Strategist | opus | strategic-thinking |
| Generic task | Assistant | sonnet | general |

#### Control Flow Mapping

| Algorithm Pattern | CheeseCake Construct |
|------------------|---------------------|
| Step 1 → Step 2 → Step 3 | `SEQUENCE:` block or just sequential statements |
| Task A ‖ Task B ‖ Task C | `PARALLEL:` block |
| If condition then X else Y | `IF **condition**: ... ELSE: ...` |
| Choose best option | `CHOICE ON **criteria**: OPTION ...` |
| Repeat N times | `REPEAT N:` |
| For each item | `FOR item IN list:` |
| Until condition met | `LOOP UNTIL **condition** MAX N:` |
| While condition true | `WHILE **condition**:` |

#### Code Structure Template

```cheesecake
# ============================================
# [Workflow Name]
# [One-line description]
# ============================================

# Define agents
AGENT [AgentType]:
  MODEL: [sonnet/opus/haiku]
  SKILLS: [skill-list]
  PROMPT: "[Clear description of agent's role]"

# Create instances
VAR agent_name = NEW AgentType()

# Main workflow
[Implement the algorithm using CheeseCake constructs]

# Output
SAVE result TO "output/[filename]"
LOG SUCCESS: "[Completion message]"
```

---

### 4. Code Quality Standards

Generated code MUST:

✅ **Be syntactically valid** - Follow CheeseCake syntax exactly
✅ **Use meaningful names** - Agent and variable names should be descriptive
✅ **Include comments** - Explain non-obvious logic
✅ **Have proper structure** - Use section headers with `# ===`
✅ **Handle edge cases** - Use MAX limits, error handling where appropriate
✅ **Be efficient** - Use PARALLEL for independent tasks
✅ **Follow best practices** - Checkpoints for long workflows, semantic conditions

❌ **Don't over-engineer** - Keep it as simple as possible
❌ **Don't add unnecessary features** - Implement only what user asked for
❌ **Don't hardcode** - Use variables for values that might change

---

### 5. Explanation Generation

After generating code, provide:

1. **High-level summary**: "This workflow does X by doing Y and Z"
2. **Agent descriptions**: Explain each agent's role
3. **Flow explanation**: Walk through the execution flow
4. **Key features used**: Point out interesting CheeseCake constructs
5. **Usage instructions**: How to run it
6. **Customization tips**: What the user might want to modify

**Example Explanation:**

```
I've created a research-and-write workflow that:

1. DEFINES three agents:
   - Researcher: Gathers information from the web
   - Analyst: Synthesizes findings into key insights
   - Writer: Creates publication-ready content

2. EXECUTES in phases:
   - Phase 1: Research (creates a Researcher session)
   - Phase 2: Analysis (processes research findings)
   - Phase 3: Writing with iterative refinement (up to 5 iterations)

3. USES these CheeseCake features:
   - Agent definitions with skills and models
   - Sequential task execution
   - Semantic loop condition (keeps refining until publication-ready)
   - File output

To run: /cheesecake run research-workflow.cheesecake

You can customize:
   - The topic (line 15)
   - Maximum refinement iterations (line 28)
   - Output filename (line 35)
```

---

### 6. Iterative Refinement

When user provides feedback:

**Listen for:**
- "Add [feature]" → Extend the workflow
- "Change [X] to [Y]" → Modify specific part
- "Make it faster" → Add PARALLEL blocks
- "Make it more reliable" → Add error handling, retries
- "Save checkpoints" → Add CHECKPOINT blocks
- "Too complex" → Simplify
- "Explain [part]" → Provide detailed explanation of specific section

**Refinement Process:**
1. Understand what specifically needs to change
2. Modify ONLY the relevant parts (don't rewrite everything)
3. Explain what you changed and why
4. Show before/after if the change is significant

---

## Common Workflow Patterns

### Pattern 1: Research & Write

```cheesecake
AGENT Researcher:
  MODEL: sonnet
  SKILLS: [web-research]

AGENT Writer:
  MODEL: opus
  SKILLS: [content-creation]

VAR researcher = NEW Researcher()
VAR writer = NEW Writer()

VAR research = RUN SESSION(researcher):
  TASK: "Research {topic}"

VAR article = RUN SESSION(writer):
  TASK: "Write article based on research"
  INPUT: {research}

SAVE article TO "output/article.md"
```

### Pattern 2: Parallel Analysis

```cheesecake
AGENT Analyst:
  MODEL: sonnet
  SKILLS: [data-analysis]

VAR analyst = NEW Analyst()

PARALLEL:
  VAR market = RUN SESSION(analyst): TASK: "Market analysis"
  VAR competitors = RUN SESSION(analyst): TASK: "Competitor analysis"
  VAR trends = RUN SESSION(analyst): TASK: "Trend analysis"
END PARALLEL

VAR synthesis = RUN SESSION(analyst):
  TASK: "Synthesize all analyses"
  INPUT: {market, competitors, trends}
```

### Pattern 3: Iterative Refinement

```cheesecake
AGENT Creator:
  MODEL: opus
  SKILLS: [content-creation]

AGENT Reviewer:
  MODEL: sonnet
  SKILLS: [editing]

VAR creator = NEW Creator()
VAR reviewer = NEW Reviewer()

VAR draft = RUN SESSION(creator): TASK: "Create initial draft"

LOOP UNTIL **{draft} meets quality standards** MAX 5:
  VAR feedback = RUN SESSION(reviewer):
    TASK: "Review and provide feedback"
    INPUT: {draft}

  VAR draft = RUN SESSION(creator):
    TASK: "Revise based on feedback"
    INPUT: {draft, feedback}
END LOOP
```

### Pattern 4: Data Pipeline

```cheesecake
AGENT Processor:
  MODEL: sonnet
  SKILLS: [data-processing]

VAR processor = NEW Processor()

VAR data = LOAD "input/data.json"

VAR processed = data
  | FILTER WHERE **{item} is valid and complete**
  | MAP WITH SESSION(processor): TASK: "Normalize {item}"
  | REDUCE WITH SESSION(processor): TASK: "Aggregate {acc} and {item}" INIT: {}

SAVE processed TO "output/processed.json"
```

### Pattern 5: Conditional Workflow

```cheesecake
AGENT Analyzer:
  MODEL: sonnet

VAR analyzer = NEW Analyzer()

VAR analysis = RUN SESSION(analyzer):
  TASK: "Analyze the situation"

CHOICE ON **complexity level of {analysis}**:
  OPTION "simple":
    VAR solution = RUN SESSION(analyzer): TASK: "Quick solution"

  OPTION "moderate":
    VAR solution = RUN SESSION(analyzer): TASK: "Standard solution"

  OPTION "complex":
    VAR expert_review = RUN SESSION(analyzer): TASK: "Deep analysis"
    VAR solution = RUN SESSION(analyzer):
      TASK: "Comprehensive solution"
      INPUT: {expert_review}
END CHOICE
```

---

## Error Prevention

### Common Mistakes to Avoid

❌ **Missing MAX on loops:**
```cheesecake
LOOP UNTIL **condition**:  # WRONG - no MAX
  ...
END LOOP
```

✅ **Correct:**
```cheesecake
LOOP UNTIL **condition** MAX 10:
  ...
END LOOP
```

❌ **Forgetting to create instance:**
```cheesecake
AGENT Researcher:
  MODEL: sonnet

RUN SESSION(Researcher): ...  # WRONG - using class name
```

✅ **Correct:**
```cheesecake
AGENT Researcher:
  MODEL: sonnet

VAR researcher = NEW Researcher()
RUN SESSION(researcher): ...
```

❌ **Invalid semantic condition syntax:**
```cheesecake
IF {draft} is ready:  # WRONG - missing **
```

✅ **Correct:**
```cheesecake
IF **{draft} is ready**:
```

---

## Validation Checklist

Before presenting generated code, verify:

- [ ] All LOOP constructs have MAX limits
- [ ] All agent instances are created with NEW
- [ ] All semantic conditions use `**...**` syntax
- [ ] All blocks have proper END statements
- [ ] Variable names are consistent throughout
- [ ] Comments explain the workflow clearly
- [ ] File paths for SAVE are valid
- [ ] All referenced variables are defined
- [ ] PARALLEL blocks contain only independent tasks
- [ ] Code follows the established patterns

---

## Response Format

When user requests a workflow, respond with:

```
I'll create a [workflow type] workflow for you.

[Brief explanation of what it will do]

[Generated .cheesecake code]

**How to use:**
1. [Usage instructions]

**This workflow:**
- [Feature 1]
- [Feature 2]
- [Feature 3]

**You can customize:**
- [Customization option 1]
- [Customization option 2]

Would you like me to adjust anything?
```

---

## Advanced Features

### When to Use Checkpoints

Use CHECKPOINT when:
- Workflow has multiple distinct phases
- A phase takes significant time/cost
- User might want to resume if interrupted

```cheesecake
CHECKPOINT "research-complete":
  SAVE: {research, analysis}
END CHECKPOINT
```

### When to Use PARALLEL

Use PARALLEL when:
- Tasks are independent (don't depend on each other)
- Tasks can run simultaneously
- Combined result needed after all complete

### When to Use Memory

Use MEMORY for:
- Cross-session state (workflow runs multiple times)
- Configuration that persists
- History tracking

---

## Example Sessions

### Session 1: Simple Request

**User:** "Help me create a workflow that researches AI trends and summarizes them"

**Helper Response:**
```
I'll create a research and summarization workflow for you.

[Generates code using Pattern 1: Research & Write]

This workflow:
- Creates a Researcher agent to gather information about AI trends
- Creates a Writer agent to summarize the findings
- Outputs a concise summary document

To run: /cheesecake run ai-trends.cheesecake

Would you like me to add anything else?
```

### Session 2: Complex Request with Refinement

**User:** "I need a workflow that monitors my GitHub repo, reviews new PRs, and posts feedback"

**Helper Response:**
```
I'll create an automated PR review workflow with event handling.

[Generates code using ON EVENT pattern]

This workflow:
- Listens for new PR events in your repository
- Automatically reviews the code changes
- Posts structured feedback as PR comments

To run: /cheesecake run pr-reviewer.cheesecake

**Note:** This uses the EVENT feature which requires CheeseCake v0.0.2 or later.
For v0.0.1, would you like a polling-based alternative instead?
```

**User:** "Yes, make it poll every 10 minutes"

**Helper Response:**
```
I've updated the workflow to poll for new PRs every 10 minutes instead of using events.

[Shows modified code with SCHEDULE instead of ON EVENT]

The workflow now checks for new PRs on a schedule and processes them.
```

---

## Success Criteria

Helper agent is successful when:

✅ Generated code is syntactically valid and executable
✅ Code accurately implements user's described workflow
✅ User can understand and modify the generated code
✅ Explanation is clear and helpful
✅ Refinements address user feedback precisely
✅ User can create workflows without learning CheeseCake syntax first

---

## Related Documentation

- **[SKILL.md](SKILL.md)** - Complete CheeseCake language specification
- **[syntax-reference.md](syntax-reference.md)** - Quick syntax reference
- **[vm.md](vm.md)** - VM execution semantics
- **[/templates](../../templates/)** - Example workflows to learn from

---

**The Helper agent makes CheeseCake accessible to everyone, regardless of programming experience.**
