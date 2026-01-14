# Interactive Dev Workflow Plugin

## Overview

A Claude Code plugin that provides an interactive development workflow. The workflow guides users through planning, implementation, and verification cycles until acceptance criteria are met.

## Core Concept

1. User triggers `/interactive-dev [task description]`
2. Main agent interviews user to gather requirements (using AskUserQuestion for multi-select UI)
3. Main agent defines "done" criteria with user
4. Task spec is written to file
5. Coding agent implements the feature
6. Verification agent tests via browser (Playwright MCP)
7. Loop continues until all done criteria pass

## Plugin Structure
```
interactive-dev-plugin/
├── .claude-plugin/
│   └── plugin.json
├── commands/
│   └── interactive-dev.md
├── agents/
│   ├── coding-agent.md
│   └── verification-agent.md
├── skills/
│   ├── planning/
│   │   └── SKILL.md
│   └── done-criteria/
│       └── SKILL.md
├── hooks/
│   └── hooks.json
├── .mcp.json
└── README.md
```

## Component Specifications

### 1. Plugin Manifest (`.claude-plugin/plugin.json`)
```json
{
  "name": "interactive-dev",
  "version": "1.0.0",
  "description": "Interactive development workflow with planning, coding, and browser-based verification loops",
  "author": {
    "name": ""
  },
  "keywords": ["workflow", "tdd", "verification", "playwright", "planning"]
}
```

### 2. Slash Command (`commands/interactive-dev.md`)

Entry point for the workflow. This runs in the main agent (not a subagent) so it can use AskUserQuestion.

**Responsibilities:**
- Load the planning skill
- Interview user with AskUserQuestion (multi-select UI)
- Define done criteria with user
- Create timestamped task file in `.claude/tasks/`
- Update `.claude/tasks/index.md`
- Orchestrate the coding → verification loop
- Evaluate completion and loop back if needed

**Task File Location:** `.claude/tasks/YYYY-MM-DD-HHMM-[slug].md`

**Task File Format:**
```markdown
# Task: [Title]

## Requirements
- [Requirement 1]
- [Requirement 2]

## Done Criteria
- [ ] [Criterion 1]
- [ ] [Criterion 2]

## Technical Decisions
- [Decision 1]
- [Decision 2]

## Files Changed
(Updated by coding agent)
```

**Loop Logic:**
1. Invoke coding-agent with task file path
2. Invoke verification-agent with task file path
3. Read updated task file
4. If all criteria checked → mark complete, exit
5. If unchecked criteria remain → pass issues to coding-agent, goto step 1

### 3. Coding Agent (`agents/coding-agent.md`)

**Frontmatter:**
```yaml
name: coding-agent
description: Implements features based on task specifications. Writes code and fixes build/lint/typecheck errors until clean. Use when implementing or fixing code for an interactive-dev task.
tools: Read, Write, Edit, Bash, Glob, Grep
model: sonnet
```

**System Prompt Responsibilities:**
- Read task file from provided path
- Understand requirements and done criteria
- Write code to implement the feature
- Run build, lint, typecheck commands (read from CLAUDE.md)
- Fix any errors (internal loop until clean)
- Update "Files Changed" section in task file
- Return summary of changes made

**Key Instructions:**
- Do NOT start the dev server
- Do NOT run browser tests
- Focus only on writing clean, working code
- Stop when build/lint/typecheck pass

### 4. Verification Agent (`agents/verification-agent.md`)

**Frontmatter:**
```yaml
name: verification-agent
description: Verifies implemented features against done criteria using automated tests and browser testing. Use for QA verification of interactive-dev tasks.
tools: Read, Edit, Bash, Glob, Grep
model: sonnet
```

**System Prompt Responsibilities:**
- Read task file from provided path
- Read dev server config from CLAUDE.md (command, port)
- Kill any existing process on the dev server port
- Start dev server fresh (ensures latest code)
- Wait for server ready
- Run automated test suite if configured
- Use Playwright MCP for browser testing:
  - Navigate to relevant pages
  - Interactive testing (click, type, verify) for complex criteria
  - Screenshot verification for visual criteria
- Check each done criterion
- Update checkboxes in task file (mark passing criteria)
- Kill dev server when done
- Return summary of results (what passed, what failed, why)

**Key Instructions:**
- Do NOT look at code implementation details
- Only receive: task file, list of changed file names
- Test behavior, not implementation
- Be thorough - test happy path and edge cases
- Take screenshots as evidence

**Fresh Eyes Principle:**
The verification agent should NOT receive:
- Code diffs
- Coding agent's reasoning
- Implementation details

It should only know:
- What to test (done criteria)
- Where to test (URLs, from CLAUDE.md)
- What files were touched (names only)

### 5. Planning Skill (`skills/planning/SKILL.md`)

**Frontmatter:**
```yaml
name: planning
description: Guides interactive requirement gathering for development tasks. Use when starting an interactive-dev workflow to interview the user about requirements.
```

**Skill Content:**
- How to break down a feature request into questions
- Question categories: scope, data, UI/UX, edge cases, technical constraints
- Each question should have:
  - Clear question text
  - 3-5 options including a recommended choice
  - "Other (specify)" option when appropriate
- Question flow: start broad, get specific
- When to stop asking (enough clarity to define done criteria)
- Output format for requirements section

### 6. Done Criteria Skill (`skills/done-criteria/SKILL.md`)

**Frontmatter:**
```yaml
name: done-criteria
description: Helps define testable acceptance criteria for development tasks. Use when establishing done criteria during interactive-dev planning.
```

**Skill Content:**
- What makes a good done criterion:
  - Observable (can be seen in browser or test output)
  - Specific (not vague or subjective)
  - Testable (can be verified programmatically or visually)
- Categories of criteria:
  - Functional (feature works)
  - Visual (UI renders correctly)
  - Validation (errors handled)
  - Build (no errors, no warnings)
- How to phrase criteria as checkable items
- Examples of good vs bad criteria:
  - Bad: "Profile page looks good"
  - Good: "Profile page displays user name, email, and avatar"
- Typical criteria count: 5-10 per task

### 7. Hooks (`hooks/hooks.json`)
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'File changed, lint/typecheck will run in coding agent'"
          }
        ]
      }
    ]
  }
}
```

Note: Keeping hooks minimal. The coding agent handles build/lint/typecheck as part of its flow.