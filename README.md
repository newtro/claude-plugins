# Mathew's Claude Plugins

A collection of Claude Code plugins for development workflows.

## Installation

Add this marketplace to Claude Code:

```
/plugin marketplace add newtro/claude-plugins
```

Then browse available plugins via `/plugin` → **Discover** tab, or install directly:

```
/plugin install <plugin-name>@mathews-claude-plugins
```

## Available Plugins

### interactive-dev

Interactive development workflow with planning, coding, and browser-based verification loops.

**Install:**
```
/plugin install interactive-dev@mathews-claude-plugins
```

**Usage:**
```
/interactive-dev [task description]
```

**Features:**
- Interview-style requirement gathering with multi-choice questions
- Testable acceptance criteria definition
- Automated implementation loop: coding agent → verification agent → fix cycle
- Playwright MCP integration for browser-based testing

[View full documentation](./interactive-dev/README.md)

## Adding New Plugins

1. Create a new folder at the repo root with your plugin structure
2. Add an entry to `.claude-plugin/marketplace.json`
3. Commit and push

**Plugin structure:**
```
my-plugin/
├── .claude-plugin/
│   └── plugin.json       # Required manifest
├── commands/             # Slash commands
├── agents/               # Subagents
├── skills/               # Agent skills
├── hooks/                # Event handlers
└── README.md
```

## License

MIT
