---
description: Explain what a .cheesecake file does
allowed-tools: Read
---

# Usage

```
/cheesecake explain <filename>
```

# What It Does

1. Reads the .cheesecake file
2. Analyzes the workflow
3. Explains in plain English:
   - What agents are defined
   - What the workflow does step-by-step
   - What inputs/outputs are expected
   - Estimated complexity (sessions, time)

# Output Example

```
📖 Explanation: research-workflow.cheesecake

**Purpose**: Research a topic and write an article

**Agents Defined**:
  • Researcher (sonnet) - Gathers information
  • Writer (opus) - Creates content

**Workflow**:
  1. Parallel Research Phase
     - Research academic papers
     - Research industry developments
     - Analyze market trends

  2. Synthesis
     - Combine all research findings

  3. Writing (iterative, max 5 loops)
     - Write draft
     - Get editor feedback
     - Revise until publication-ready

  4. Output
     - Save final article to file

**Estimated**:
  • Sessions: 5-12 (depends on loop iterations)
  • Time: 2-5 minutes
  • Cost: ~$0.05-0.15
```
