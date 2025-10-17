# Plugin Usage Guide

## Installation from GitHub

### Step 1: Add the Marketplace

```bash
/plugin marketplace add huangmingxia/test-generation-assistant
```

This adds your GitHub repository as a plugin marketplace to Claude Code.

### Step 2: Install the Plugin

```bash
/plugin install test-gen
```

Or browse and install from the plugin menu:
```bash
/plugin
```

Then select `test-gen` from the list.

## Using the Plugin

Once installed, use the plugin commands with the `test-gen:` prefix:

```bash
/test-gen:full-workflow HIVE-2883
/test-gen:generate-test HIVE-2883
/test-gen:generate-e2e HIVE-2883
/test-gen:run-tests HIVE-2883
/test-gen:submit-pr HIVE-2883
```

## Available Commands

| Command | Description | Duration |
|---------|-------------|----------|
| `/test-gen:full-workflow` | Execute complete workflow | ~3-4 min |
| `/test-gen:generate-test` | Generate test cases from JIRA | ~90s |
| `/test-gen:generate-e2e` | Generate E2E test code | ~120s |
| `/test-gen:run-tests` | Execute E2E tests | ~180s |
| `/test-gen:submit-pr` | Create pull request | ~30s |
| `/test-gen:generate-report` | Generate comprehensive report | ~45s |
| `/test-gen:regenerate-test` | Force regenerate test case | ~90s |
| `/test-gen:regenerate-e2e` | Force regenerate E2E code | ~120s |

## Quick Start

### Complete Workflow
```bash
# One command does everything
/test-gen:full-workflow HIVE-2883
```

### Step-by-Step
```bash
# 1. Generate test cases
/test-gen:generate-test HIVE-2883

# 2. Generate E2E code
/test-gen:generate-e2e HIVE-2883

# 3. Run tests
/test-gen:run-tests HIVE-2883

# 4. Generate report
/test-gen:generate-report HIVE-2883

# 5. Submit PR
/test-gen:submit-pr HIVE-2883
```

## Local Development

If you're developing this plugin locally (not installing from GitHub):

1. **Open the project in Claude Code**
   ```bash
   cd /path/to/test-generation-assistant
   code .
   ```

2. **Commands are automatically available**
   - Root commands: `/full-workflow`, `/generate-test-case`, etc.
   - Plugin commands: `/test-gen:full-workflow`, `/test-gen:generate-test`, etc.

No installation needed for local development!

## Managing Plugins

### List Installed Plugins
```bash
/plugin
```

### Update Plugin
```bash
/plugin update test-gen
```

### Uninstall Plugin
```bash
/plugin uninstall test-gen
```

### Remove Marketplace
```bash
/plugin marketplace remove test-generation-assistant
```

## Plugin Configuration

The plugin is configured via `.claude-plugin/marketplace.json`:

```json
{
  "name": "test-generation-assistant",
  "owner": {
    "name": "huangmingxia"
  },
  "plugins": [
    {
      "name": "test-gen",
      "source": "github:huangmingxia/test-generation-assistant/plugins/test-gen",
      "description": "AI-assisted test generation and execution system for OpenShift components"
    }
  ]
}
```

The plugin itself is configured via `plugins/test-gen/.claude-plugin/plugin.json`:

```json
{
  "name": "test-gen",
  "version": "1.0.0",
  "description": "OpenShift Hive AI Test Generation System",
  "author": "OpenShift QE Team",
  "contextFiles": [
    "CLAUDE.md",
    "STARTUP_CHECKLIST.md",
    "../../config/agents/**/*.md"
  ]
}
```

## Prerequisites

Ensure configured:
- ✅ JIRA MCP server
- ✅ GitHub CLI (`gh`) authenticated
- ✅ OpenShift cluster kubeconfig
- ✅ openshift-tests-private fork

## Command Examples

### Example 1: New JIRA Issue
```bash
# Complete workflow
/test-gen:full-workflow HIVE-2883

# Then submit PR
/test-gen:submit-pr HIVE-2883
```

### Example 2: Update Existing Test
```bash
# Regenerate test case
/test-gen:regenerate-test HIVE-2883

# Regenerate E2E code
/test-gen:regenerate-e2e HIVE-2883

# Run tests
/test-gen:run-tests HIVE-2883
```

### Example 3: Manual Control
```bash
# Step 1: Generate test cases
/test-gen:generate-test HIVE-2883

# Review output in test_artifacts/

# Step 2: Generate E2E code
/test-gen:generate-e2e HIVE-2883

# Review code in temp_repos/

# Step 3: Run tests
/test-gen:run-tests HIVE-2883

# Review results

# Step 4: Submit PR
/test-gen:submit-pr HIVE-2883
```

## Output Structure

```
test_artifacts/{COMPONENT}/{JIRA_KEY}/
├── phases/
│   ├── test_requirements_output.md
│   ├── test_strategy.md
│   └── comprehensive_test_results.md
├── test_cases/
│   └── {JIRA_KEY}_test_cases.md
├── test_coverage_matrix.md
└── test_report.md

temp_repos/openshift-tests-private/
└── test/extended/{component}/
    └── {jira_key}_test.go
```

## Troubleshooting

### Plugin Not Installing

**Issue:** `/plugin install test-gen` fails

**Solutions:**
1. Ensure marketplace is added:
   ```bash
   /plugin marketplace add huangmingxia/test-generation-assistant
   ```
2. Verify GitHub repository is public
3. Check internet connection
4. Try installing from the `/plugin` menu instead

### Commands Not Recognized

**Issue:** `/test-gen:xxx` commands not available

**Solutions:**
1. List installed plugins:
   ```bash
   /plugin
   ```
2. Reinstall if needed:
   ```bash
   /plugin uninstall test-gen
   /plugin install test-gen
   ```
3. Restart Claude Code

### Agent Not Found

**Issue:** "Agent config not found" error

**Solution:**
- This is a plugin configuration issue
- The plugin references `../../config/agents/` which should exist in the repository
- File an issue on GitHub if this persists

## Plugin vs Root Commands

This repository provides both plugin commands and root commands:

| Plugin Command | Root Command | Note |
|----------------|--------------|------|
| `/test-gen:full-workflow` | `/full-workflow` | Plugin: for installed users<br>Root: for local development |
| `/test-gen:generate-test` | `/generate-test-case` | Same functionality |
| `/test-gen:generate-e2e` | `/generate-e2e-case` | Same functionality |
| `/test-gen:run-tests` | `/run-tests` | Same functionality |
| `/test-gen:submit-pr` | `/submit-pr` | Same functionality |
| `/test-gen:generate-report` | `/generate-report` | Same functionality |
| `/test-gen:regenerate-test` | `/regenerate-test` | Same functionality |
| `/test-gen:regenerate-e2e` | `/regenerate-e2e` | Same functionality |

**When to use which:**
- **Plugin commands** (`/test-gen:*`): Use when you installed the plugin from GitHub
- **Root commands** (`/*`): Use when working on the repository locally

Both execute the same agents and produce identical results.

## Documentation

- **Slash Commands Guide**: `SLASH_COMMANDS.md` - Detailed documentation for root commands
- **Plugin Commands**: `plugins/test-gen/commands/README.md`
- **Agent Configurations**: `config/agents/`
- **Workflow Guide**: `WORKFLOW_ORCHESTRATOR_GUIDE.md`
- **Startup Checklist**: `STARTUP_CHECKLIST.md`

## Publishing Updates

If you're maintaining this plugin:

1. **Make changes to plugin code**
2. **Update version** in `plugins/test-gen/.claude-plugin/plugin.json`
3. **Commit and push**:
   ```bash
   git add .
   git commit -m "Update test-gen plugin v1.1.0"
   git push origin main
   ```
4. **Users auto-update** or manually update:
   ```bash
   /plugin update test-gen
   ```

## More Information

- **Official Plugin Documentation**: https://docs.claude.com/en/docs/claude-code/plugins
- **Claude Code Plugins Announcement**: https://www.anthropic.com/news/claude-code-plugins
- **Repository**: https://github.com/huangmingxia/test-generation-assistant

## Example Marketplaces

Learn from other plugin marketplaces:
- Anthropic's official plugins: `/plugin marketplace add anthropics/claude-code`
- Community examples in the [Claude Code Plugins announcement](https://www.anthropic.com/news/claude-code-plugins)
