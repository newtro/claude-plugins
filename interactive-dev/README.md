# Interactive Dev Plugin

A Claude Code plugin that provides an interactive development workflow with planning, implementation, and browser-based verification loops.

## Overview

This plugin guides you through a structured development process:

1. **Planning**: Interview-style requirement gathering using multi-choice questions
2. **Criteria Definition**: Establish testable acceptance criteria
3. **Implementation Loop**: Automated code → verify → fix cycle until all criteria pass

## Installation

1. Clone or copy this plugin to your Claude Code plugins directory
2. The plugin will be automatically discovered by Claude Code

## Usage

```
/interactive-dev [task description]
```

### Example

```
/interactive-dev Add a user login form with email/password authentication
```

## How It Works

### Phase 1: Planning Interview

The plugin asks targeted questions to understand your requirements:
- What's the feature scope?
- What UI/UX patterns to follow?
- How to handle edge cases?

Questions use multi-choice format with recommended options.

### Phase 2: Done Criteria

Based on your answers, the plugin proposes testable acceptance criteria:
- Functional requirements (feature works)
- Visual requirements (UI renders correctly)
- Validation requirements (errors handled)
- Build requirements (no errors)

### Phase 3: Implementation Loop

```
┌─────────────────┐
│  Coding Agent   │──────────────┐
│  (implement)    │              │
└────────┬────────┘              │
         │                       │
         ▼                       │
┌─────────────────┐              │
│ Verification    │              │
│ Agent (test)    │              │
└────────┬────────┘              │
         │                       │
         ▼                       │
    ┌────────────┐               │
    │ All pass?  │───No──────────┘
    └─────┬──────┘
          │ Yes
          ▼
       Complete
```

The loop continues until all criteria pass (max 5 iterations).

## Plugin Structure

```
interactive-dev/
├── .claude-plugin/
│   └── plugin.json        # Plugin manifest
├── commands/
│   └── interactive-dev.md # Main slash command
├── agents/
│   ├── coding-agent.md    # Implementation subagent
│   └── verification-agent.md # QA verification subagent
├── skills/
│   ├── planning/
│   │   └── SKILL.md       # Requirement gathering guidance
│   └── done-criteria/
│       └── SKILL.md       # Acceptance criteria guidance
├── hooks/
│   └── hooks.json         # Hook configuration
├── .mcp.json              # Playwright MCP config
└── README.md
```

## Task Files

Tasks are tracked in `.claude/tasks/` with timestamped filenames:

```
.claude/tasks/2024-01-15-1430-add-login-form.md
```

Each task file contains:
- Requirements
- Done criteria (with checkboxes)
- Technical decisions
- Files changed

## Requirements

### Project Configuration

Your project's `CLAUDE.md` should include:

```markdown
## Build Commands
npm run build
npm run lint
npm run typecheck

## Dev Server
npm run dev
# Port: 3000
# URL: http://localhost:3000
```

### Dependencies

The verification agent uses Playwright MCP for browser testing. The `.mcp.json` file configures this automatically.

## Agents

### Coding Agent

Implements features based on task specifications:
- Reads requirements and done criteria
- Writes clean code following project patterns
- Runs build/lint/typecheck until clean
- Does NOT start dev server or run browser tests

### Verification Agent

Tests implementations against criteria:
- Fresh eyes approach (doesn't see code implementation)
- Manages dev server lifecycle
- Uses Playwright for browser testing
- Updates checkboxes in task file

## Skills

### Planning Skill

Guides requirement interviews:
- Question categories (scope, data, UI/UX, edge cases)
- Question structure with recommendations
- When to stop asking questions

### Done Criteria Skill

Helps define testable criteria:
- Observable, specific, testable requirements
- Categories: functional, visual, validation, build
- Examples of good vs bad criteria

## Configuration

### hooks.json

Minimal hooks that notify when files change. The coding agent handles build/lint/typecheck as part of its workflow.

### .mcp.json

Configures Playwright MCP server for browser-based testing:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-playwright"]
    }
  }
}
```

## License

MIT
