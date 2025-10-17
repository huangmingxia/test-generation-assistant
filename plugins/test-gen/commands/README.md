# Slash Commands Guide

## Available Commands

### Core Workflow Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `/generate-test` | Generate test case from JIRA | `/generate-test HIVE-2883` |
| `/generate-e2e` | Generate E2E test code | `/generate-e2e HIVE-2883` |
| `/run-tests` | Execute E2E tests | `/run-tests HIVE-2883` |
| `/generate-report` | Generate comprehensive test report | `/generate-report HIVE-2883` |
| `/submit-pr` | Create pull request | `/submit-pr HIVE-2883` |
| `/full-workflow` | Execute complete workflow | `/full-workflow HIVE-2883` |

### Regeneration Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `/regenerate-test` | Regenerate test case (force) | `/regenerate-test HIVE-2883` |
| `/regenerate-e2e` | Regenerate E2E code (force) | `/regenerate-e2e HIVE-2883` |

## Command vs Agent Relationship

```
┌─────────────────────┐
│  Slash Command      │  ← User types this
│  /generate-test     │
└──────────┬──────────┘
           │
           ↓
┌─────────────────────┐
│  Agent Config       │  ← Claude reads this
│  test_case_         │
│  generation.yaml    │
└──────────┬──────────┘
           │
           ↓
┌─────────────────────┐
│  Agent Execution    │  ← Claude executes steps
│  Step 1, 2, 3...    │
└─────────────────────┘
```

## How Slash Commands Work

### 1. User Interface Layer (Slash Command)
- **File**: `.claude/commands/generate-test.md`
- **Purpose**: User-friendly entry point
- **Contains**:
  - Usage instructions
  - Argument parsing
  - Execution prompt

### 2. Logic Layer (Agent)
- **File**: `config/agents/test_case_generation.yaml`
- **Purpose**: Execution logic and steps
- **Contains**:
  - Step-by-step tasks
  - Tool configurations
  - Input/output specifications

### 3. Execution Flow

```bash
# User types
/generate-test HIVE-2883

# Claude Code processes
1. Reads .claude/commands/generate-test.md
2. Extracts argument: HIVE-2883
3. Reads config/agents/test_case_generation.yaml
4. Executes each step in agent config
5. Returns results to user
```

## Creating Your Own Slash Command

### Template Structure

```markdown
# Command Title

Brief description of what this command does.

## Usage
\`\`\`
/your-command ARG1 ARG2
\`\`\`

## What this command does
1. Step 1 description
2. Step 2 description
3. ...

## Arguments
- `$1` (required): Description
- `$2` (optional): Description

## Example
\`\`\`
User: /your-command value1 value2
→ Expected behavior
→ Output description
\`\`\`

---

Execute {agent_name} agent for: **{args}**
```

### Key Components

1. **Header**: Clear command name and description
2. **Usage**: Show command syntax
3. **What it does**: High-level workflow
4. **Arguments**: Define required/optional params
5. **Example**: Show concrete usage
6. **Footer**: Execution prompt (critical!)

### The Footer Pattern

```markdown
---

Execute {agent_name} agent for: **{args}**
```

This footer tells Claude Code:
- Which agent to load (`{agent_name}`)
- What arguments to pass (`{args}`)

## Best Practices

### DO ✅
- Keep commands simple and focused
- Document all arguments clearly
- Provide concrete examples
- Link to agent configuration
- Use consistent naming patterns

### DON'T ❌
- Create commands without corresponding agents
- Mix multiple agents in one command
- Skip argument documentation
- Forget the execution footer

## Common Patterns

### Pattern 1: Simple Agent Execution
```markdown
Execute test_case_generation agent for: **{args}**
```

### Pattern 2: Conditional Execution
```markdown
Check prerequisites, then execute e2e_generation agent for: **{args}**
```

### Pattern 3: Sequential Workflow
```markdown
Execute workflow: test_case_generation → e2e_generation → test-executor for: **{args}**
```

## Troubleshooting

### Command not found
- Check file exists in `.claude/commands/`
- Verify file has `.md` extension
- Ensure file name matches command (e.g., `generate-test.md` → `/generate-test`)

### Command doesn't execute correctly
- Verify agent config exists
- Check execution footer format
- Ensure argument parsing is correct

### Agent not loading
- Verify agent path in `config/agents/`
- Check agent YAML is valid
- Ensure agent name in command matches file name

## Advanced: Multi-Step Commands

For complex workflows, use sequential agent calls:

```markdown
Execute complete workflow for: **{args}**

MANDATORY steps:
1. Read config/agents/workflow_orchestrator.yaml
2. Load test_case_generation agent
3. Execute all 4 phases
4. Verify outputs before proceeding
```

## Related Documentation

- Agent configurations: [config/agents/](../../config/agents/)
- Workflow guide: [prompts/workflow_guide.md](../../prompts/workflow_guide.md)
- Execution checklist: [prompts/check_list.md](../../prompts/check_list.md)
