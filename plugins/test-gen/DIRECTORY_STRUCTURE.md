# Plugin Directory Structure

## Overview

This plugin follows Claude Plugin standards with both `.claude/commands/` and `commands/` directories serving different purposes.

## Directory Structure

```
plugins/test-gen/
├── .claude-plugin/                       ← Plugin metadata
│   └── plugin.json                       ← Plugin configuration
│
├── commands/                             ← Plugin command documentation
│   ├── full-workflow.md                  ← Command reference docs
│   ├── generate-test.md
│   └── ...
│
├── CLAUDE.md                             ← Agent execution context
├── STARTUP_CHECKLIST.md                  ← Startup checklist
└── OUTPUT_STRUCTURE.md                   ← Output structure docs

../../config/                             ← Shared configurations (root)
├── agents/                               ← Agent definitions
│   ├── test_case_generation.md
│   ├── e2e_test_generation_openshift_private.md
│   └── ...
├── rules/                                ← Test generation rules
└── templates/                            ← Output templates
```

## Directory Purposes

### `.claude-plugin/` - Plugin Metadata

**Purpose**: Plugin configuration and metadata

**Key file**: `plugin.json` - Defines plugin name, version, context files

**Example**:
```json
{
  "name": "test-gen",
  "version": "1.0.0",
  "contextFiles": [
    "CLAUDE.md",
    "../../config/agents/**/*.md"
  ]
}
```

### `commands/` - Command Documentation

**Purpose**: Plugin command documentation and reference

**Usage**: Users type `/test-gen:command-name` to invoke

**Format**: Markdown files with YAML frontmatter

**Example**:
```markdown
---
description: Generate test cases from JIRA
argument-hint: [JIRA_KEY]
---

## Name
generate-test

## Synopsis
/test-gen:generate-test JIRA_KEY
```

**Characteristics**:
- Executable by users via `/test-gen:xxx`
- Processed by Claude Code's slash command system
- References agents in `../../config/agents/`

### `../../config/` - Shared Configuration (Root Directory)

**Purpose**: Centralized configuration shared across the entire project

**Why shared?**
- Single source of truth for all agents, rules, and templates
- Avoid duplication and sync issues
- Both root-level slash commands and plugin commands use the same config

## Command Name Format

All commands use the format: `/test-gen:command-name`

Where:
- `test-gen` = plugin name (from `.claude-plugin/plugin.json`)
- `:` = separator
- `command-name` = actual command (matches filename without `.md`)

## Available Commands

| Slash Command | File Location | Purpose |
|---------------|---------------|---------|
| `/test-gen:full-workflow` | `commands/full-workflow.md` | Complete workflow |
| `/test-gen:generate-test` | `commands/generate-test.md` | Generate test cases |
| `/test-gen:generate-e2e` | `commands/generate-e2e.md` | Generate E2E code |
| `/test-gen:run-tests` | `commands/run-tests.md` | Execute tests |
| `/test-gen:submit-pr` | `commands/submit-pr.md` | Submit PR |
| `/test-gen:regenerate-test` | `commands/regenerate-test.md` | Regenerate tests |
| `/test-gen:regenerate-e2e` | `commands/regenerate-e2e.md` | Regenerate E2E |
| `/test-gen:generate-report` | `commands/generate-report.md` | Generate report |

## Agent Execution Flow

```
User types: /test-gen:generate-test HIVE-2883
     ↓
Claude reads: commands/generate-test.md
     ↓
Frontmatter & Synopsis parsed
     ↓
Implementation section references: ../../config/agents/test_case_generation.md
     ↓
Agent executes steps
     ↓
Results returned to user
```

## Plugin Configuration

### `.claude-plugin/plugin.json`

```json
{
  "name": "test-gen",
  "version": "1.0.0",
  "contextFiles": [
    "CLAUDE.md",
    "config/agents/**/*.md"
  ]
}
```

**Key fields**:
- `name`: Plugin name (used in command prefix)
- `contextFiles`: Files Claude loads for context

## Best Practices

### DO ✅
- Use relative paths to reference root `config/` (e.g., `../../config/agents/`)
- Include frontmatter in all command files
- Document prerequisites clearly
- Keep agent configurations in root `config/` directory
- Update `.claude-plugin/plugin.json` when adding new context files

### DON'T ❌
- Duplicate config files in plugin directory
- Hard-code absolute paths
- Skip frontmatter in command files
- Create plugin-specific configs (use shared root config instead)

## Updating Commands

When updating a command:

1. Update `commands/command-name.md`
2. Test with: `/test-gen:command-name`
3. Verify agent execution works
4. Update root `config/agents/` if agent logic changes

## Configuration Sharing

### Why Not Duplicate Config?

✅ **Advantages of Shared Config**:
- Single source of truth
- No sync issues between root and plugin
- Easier maintenance
- Consistent behavior across all commands

❌ **Problems with Duplicated Config**:
- Config drift between copies
- Double maintenance burden
- Inconsistent behavior
- Wasted disk space

### Path References

All plugin commands reference root config using relative paths:
- `../../config/agents/` - Agent definitions
- `../../config/rules/` - Test generation rules
- `../../config/templates/` - Output templates

## See Also

- [Command Format Guide](commands/README.md)
- [Plugin Configuration](.claude-plugin/plugin.json)
- [Agent Documentation](../../config/agents/)
- [CLAUDE.md](CLAUDE.md) - Agent execution context
