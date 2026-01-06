---
description: Create new .cheesecake file with Helper wizard
allowed-tools: Task, Write, Read, AskUserQuestion
---

# /cheesecake create

Creates a new `.cheesecake` workflow file using the Helper agent - no syntax knowledge required!

---

## Usage

```
/cheesecake create [--template <name>]
```

**Options:**
- `--template <name>`: Create from existing template (optional)
- No args: Launch interactive Helper wizard

---

## Implementation

### Without Template (Interactive Helper)

1. **Greet the user:**
   ```
   ✨ CheeseCake Helper

   I'll help you create a workflow! Just describe what you want to accomplish in plain English.

   What would you like your workflow to do?
   ```

2. **Receive user description:**
   - Listen to their natural language description
   - Ask clarifying questions if needed (use AskUserQuestion tool)

3. **Launch Helper Agent:**
   ```
   Use the Task tool to launch the Helper agent:

   Task(
     subagent_type: "general-purpose",
     prompt: "You are the CheeseCake Helper agent. Your role is to convert natural language workflow descriptions into valid .cheesecake files.

     SKILLS: You have access to:
     - cheesecake/helper (conversion methodology)
     - cheesecake/SKILL (complete language spec)
     - cheesecake/syntax-reference (quick reference)

     USER REQUEST: {user_description}

     TASK:
     1. Analyze the user's workflow requirements
     2. Design an algorithmic approach
     3. Generate valid .cheesecake code
     4. Provide clear explanation of the code
     5. Suggest a filename

     Follow the guidelines in the Helper skill to ensure quality output."
   )
   ```

4. **Receive generated code from Helper:**
   - Helper will provide:
     - Complete .cheesecake code
     - Explanation of what it does
     - Suggested filename
     - Customization tips

5. **Save the file:**
   ```
   Use Write tool to save to suggested filename or ask user for preferred name
   ```

6. **Present to user:**
   ```
   ✓ Created: {filename}

   [Show explanation from Helper]

   Would you like to:
     [1] Run it now (/cheesecake run {filename})
     [2] View the code
     [3] Make changes (describe modifications)
     [4] Done
   ```

7. **Handle user choice:**
   - Option 1: Execute `/cheesecake run {filename}`
   - Option 2: Show the generated code
   - Option 3: Re-launch Helper with refinement request
   - Option 4: Complete

### With Template

If user provides `--template <name>`:

1. **List available templates** if no name given:
   ```
   Available templates:
     1. hello-world         - Simple introduction
     2. research-and-write  - Research topic, write article
     3. code-review         - Automated PR review
     4. data-pipeline       - Process data through stages
     5. content-moderation  - Review content for issues

   Which template? (1-5)
   ```

2. **Copy template:**
   - Read the template file from `/templates/{name}.cheesecake`
   - Ask for customization parameters (topic, output path, etc.)
   - Optionally use Helper to customize the template

3. **Save customized file:**
   - Write to new filename
   - Show what was customized

---

## Example Interactions

### Example 1: Creating from Scratch

```
User: /cheesecake create

Assistant: ✨ CheeseCake Helper

I'll help you create a workflow\! What would you like your workflow to do?

User: "Research AI trends and write a blog post"

Assistant: Great\! I'll create a research-and-write workflow.

[Helper agent generates code...]

✓ Created: ai-trends-blog.cheesecake

This workflow:
- Uses a Researcher agent to gather information about AI trends
- Uses a Writer agent to create an engaging blog post
- Saves output to output/blog-post.md

Would you like to:
  [1] Run it now
  [2] View the code
  [3] Make changes
  [4] Done

User: 1

Assistant: Running ai-trends-blog.cheesecake...
[Execution proceeds]
```

---

## Notes

- The Helper agent makes CheeseCake accessible to non-programmers
- Generated code is always syntactically valid
- User can iterate and refine without learning syntax
- Templates provide quick starting points for common patterns
