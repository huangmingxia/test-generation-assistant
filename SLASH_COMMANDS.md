# Slash Commands Usage Guide

## Overview

This project provides direct slash commands in the `.claude/commands/` directory. These commands are automatically available when you open this project in Claude Code.

**No installation required!**

## Available Commands

| Command | Description | Duration |
|---------|-------------|----------|
| `/full-workflow` | Execute complete workflow | ~3-4 min |
| `/generate-test-case` | Generate test cases from JIRA | ~90s |
| `/generate-e2e-case` | Generate E2E test code | ~120s |
| `/run-tests` | Execute E2E tests | ~180s |
| `/generate-report` | Generate comprehensive report | ~45s |
| `/submit-pr` | Create pull request | ~30s |
| `/regenerate-test` | Force regenerate test case | ~90s |
| `/regenerate-e2e` | Force regenerate E2E code | ~120s |

## Quick Start

### Complete Workflow (One Command)
```bash
/full-workflow HIVE-2883
```

This automatically executes:
1. Generate test cases
2. Generate E2E code  
3. Run tests

Total time: ~3-4 minutes

### Step-by-Step Workflow
```bash
# Step 1: Generate test cases
/generate-test-case HIVE-2883

# Step 2: Generate E2E code
/generate-e2e-case HIVE-2883

# Step 3: Run tests
/run-tests HIVE-2883

# Step 4: Generate report (optional)
/generate-report HIVE-2883

# Step 5: Submit PR
/submit-pr HIVE-2883
```

## Command Details

### `/full-workflow JIRA_KEY`

Execute complete test generation workflow.

**What it does:**
1. Generates test cases from JIRA issue
2. Creates E2E test code and integrates into openshift-tests-private
3. Executes tests in OpenShift cluster

**Prerequisites:**
- JIRA MCP server configured
- GitHub CLI authenticated
- OpenShift cluster kubeconfig

**Example:**
```bash
/full-workflow HIVE-2883
/full-workflow CCO-1234
```

**Output:**
```
test_artifacts/{COMPONENT}/{JIRA_KEY}/
├── phases/
├── test_cases/
└── test_coverage_matrix.md

temp_repos/openshift-tests-private/
└── test/extended/{component}/{jira_key}_test.go
```

---

### `/generate-test-case JIRA_KEY`

Generate test cases from JIRA issue.

**What it does:**
1. Analyzes JIRA requirements
2. Generates test strategy
3. Creates test case design
4. Builds test coverage matrix

**Example:**
```bash
/generate-test-case HIVE-2883
/generate-test-case CCO-1234
```

**Output:**
```
test_artifacts/{COMPONENT}/{JIRA_KEY}/
├── phases/
│   ├── test_requirements_output.md
│   └── test_strategy.md
├── test_cases/
│   └── {JIRA_KEY}_test_cases.md
└── test_coverage_matrix.md
```

---

### `/generate-e2e-case JIRA_KEY`

Generate E2E test code and integrate into openshift-tests-private.

**Prerequisites:**
- Test case must exist (run `/generate-test-case` first)

**What it does:**
1. Checks test case exists
2. Clones/updates openshift-tests-private repository
3. Generates E2E test code
4. Creates feature branch: `ai-case-design-{JIRA_KEY}`

**Example:**
```bash
/generate-e2e-case HIVE-2883
/generate-e2e-case CCO-1234
```

**Output:**
```
temp_repos/openshift-tests-private/
└── test/extended/{component}/
    └── {jira_key}_test.go
```

**Branch:** `ai-case-design-{JIRA_KEY}`

---

### `/run-tests JIRA_KEY`

Execute E2E tests in OpenShift cluster.

**Prerequisites:**
- E2E code must exist (run `/generate-e2e-case` first)
- OpenShift cluster kubeconfig accessible

**What it does:**
1. Locates E2E test code
2. Executes tests in cluster
3. Collects results
4. Generates execution report

**Example:**
```bash
/run-tests HIVE-2883
/run-tests CCO-1234
```

**Output:**
```
test_artifacts/{COMPONENT}/{JIRA_KEY}/phases/
└── comprehensive_test_results.md
```

---

### `/generate-report JIRA_KEY`

Generate comprehensive test report.

**Prerequisites:**
- Test artifacts must exist from previous workflow steps

**What it does:**
1. Loads report template
2. Gathers all test artifacts
3. Generates comprehensive report
4. Validates report completeness

**Example:**
```bash
/generate-report HIVE-2883
/generate-report CCO-1234
```

**Output:**
```
test_artifacts/{COMPONENT}/{JIRA_KEY}/
└── test_report.md
```

---

### `/submit-pr JIRA_KEY`

Create pull request to openshift-tests-private.

**Prerequisites:**
- E2E code must exist and be validated
- Git fork configured
- Branch `ai-case-design-{JIRA_KEY}` exists

**What it does:**
1. Verifies E2E code exists
2. Creates commit with proper format
3. Pushes to fork repository
4. Creates pull request

**Example:**
```bash
/submit-pr HIVE-2883
/submit-pr CCO-1234
```

**Output:**
- Pull request URL
- PR follows openshift-tests-private submission rules

---

### `/regenerate-test JIRA_KEY`

Force regenerate test case (skip prerequisite checks).

**Use when:**
- Fixing issues in existing test case
- Updating based on new requirements
- JIRA issue has been updated

**Difference from `/generate-test-case`:**
- Skips all prerequisite checks
- Forces regeneration even if exists
- Overwrites existing files

**Example:**
```bash
/regenerate-test HIVE-2883
/regenerate-test CCO-1234
```

---

### `/regenerate-e2e JIRA_KEY`

Force regenerate E2E code (skip prerequisite checks).

**Use when:**
- Fixing issues in E2E code
- Updating based on test case changes
- Need complete regeneration

**Difference from `/generate-e2e-case`:**
- Skips prerequisite checks
- Forces regeneration even if exists
- Overwrites existing E2E files

**Example:**
```bash
/regenerate-e2e HIVE-2883
/regenerate-e2e CCO-1234
```

## Usage Examples

### Example 1: New JIRA Issue (Full Workflow)
```bash
# One command does it all
/full-workflow HIVE-2883

# Review outputs, then submit PR
/submit-pr HIVE-2883
```

### Example 2: New JIRA Issue (Step by Step)
```bash
# Step 1: Generate test cases
/generate-test-case HIVE-2883

# Review: test_artifacts/hive/HIVE-2883/test_cases/

# Step 2: Generate E2E code
/generate-e2e-case HIVE-2883

# Review: temp_repos/openshift-tests-private/

# Step 3: Run tests
/run-tests HIVE-2883

# Review: test_artifacts/hive/HIVE-2883/phases/comprehensive_test_results.md

# Step 4: Generate report
/generate-report HIVE-2883

# Review: test_artifacts/hive/HIVE-2883/test_report.md

# Step 5: Submit PR
/submit-pr HIVE-2883
```

### Example 3: Update Existing Test
```bash
# JIRA issue updated, regenerate test case
/regenerate-test HIVE-2883

# Regenerate E2E code based on new test case
/regenerate-e2e HIVE-2883

# Run tests
/run-tests HIVE-2883

# Submit updated PR
/submit-pr HIVE-2883
```

### Example 4: Fix Test Issues
```bash
# Test case has issues, regenerate
/regenerate-test HIVE-2883

# E2E code has issues, regenerate
/regenerate-e2e HIVE-2883
```

## Prerequisites

### Required Tools
- **JIRA MCP**: Configured and accessible
- **GitHub CLI**: `gh` installed and authenticated
- **OpenShift**: Cluster kubeconfig available
- **Git Fork**: openshift-tests-private fork configured

### Environment Setup
Verify setup with startup checklist:
```bash
# View checklist
cat STARTUP_CHECKLIST.md
```

## Output Structure

### Test Artifacts
```
test_artifacts/{COMPONENT}/{JIRA_KEY}/
├── phases/
│   ├── test_requirements_output.md    # JIRA requirements analysis
│   ├── test_strategy.md               # Test strategy
│   └── comprehensive_test_results.md  # Test execution results
├── test_cases/
│   └── {JIRA_KEY}_test_cases.md      # Test case design
├── test_coverage_matrix.md            # Coverage matrix
└── test_report.md                     # Comprehensive report
```

### E2E Code
```
temp_repos/openshift-tests-private/
└── test/extended/{component}/
    └── {jira_key}_test.go             # E2E test code
```

## Workflow Flow Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                     /full-workflow                          │
│  (combines generate-test-case → generate-e2e-case → run-tests)  │
└─────────────────────────────────────────────────────────────┘
                              │
                              ↓
┌────────────────────┐    ┌────────────────────┐    ┌────────────────────┐
│ /generate-test-case│ →  │ /generate-e2e-case │ →  │    /run-tests      │
│     (90s)          │    │      (120s)        │    │      (180s)        │
└────────────────────┘    └────────────────────┘    └────────────────────┘
         │                         │                          │
         ↓                         ↓                          ↓
   Test artifacts            E2E test code            Test results
                                   │
                                   ↓
                         ┌────────────────────┐
                         │  /generate-report  │
                         │      (45s)         │
                         └────────────────────┘
                                   │
                                   ↓
                         ┌────────────────────┐
                         │    /submit-pr      │
                         │      (30s)         │
                         └────────────────────┘
                                   │
                                   ↓
                              Pull Request
```

## Troubleshooting

### Command Not Found
**Issue:** Command not recognized in Claude Code

**Solution:**
- Confirm project is open in Claude Code
- Check `.claude/commands/` directory exists
- Verify command files have correct format

### Agent Execution Failed
**Issue:** "Agent not found" or execution error

**Solution:**
- Verify `config/agents/` directory exists
- Check agent file extensions (`.md`)
- Confirm agent configuration is valid

### Test Case Generation Failed
**Issue:** `/generate-test-case` fails

**Solution:**
- Check JIRA MCP is configured
- Verify JIRA issue key is valid
- Ensure component rules exist in `config/rules/`

### E2E Code Generation Failed
**Issue:** `/generate-e2e-case` fails

**Solution:**
- Verify test case exists (prerequisite)
- Check GitHub CLI is authenticated
- Ensure openshift-tests-private fork configured

### Test Execution Failed
**Issue:** `/run-tests` fails

**Solution:**
- Verify E2E code exists
- Check OpenShift cluster kubeconfig is accessible
- Ensure cluster is reachable

## Documentation

- **Slash Commands**: `.claude/commands/README.md`
- **Agent Configurations**: `config/agents/`
- **Workflow Guide**: `WORKFLOW_ORCHESTRATOR_GUIDE.md`
- **Startup Checklist**: `STARTUP_CHECKLIST.md`
- **Plugin Usage**: `PLUGIN_USAGE.md`

## Command vs Plugin Commands

This project also provides plugin commands (with `test-gen:` prefix):

| Root Command | Plugin Command | Note |
|--------------|----------------|------|
| `/full-workflow` | `/test-gen:full-workflow` | Same functionality |
| `/generate-test-case` | `/test-gen:generate-test` | Same functionality |
| `/generate-e2e-case` | `/test-gen:generate-e2e` | Same functionality |
| `/run-tests` | `/test-gen:run-tests` | Same functionality |
| `/submit-pr` | `/test-gen:submit-pr` | Same functionality |
| `/generate-report` | `/test-gen:generate-report` | Same functionality |
| `/regenerate-test` | `/test-gen:regenerate-test` | Same functionality |
| `/regenerate-e2e` | `/test-gen:regenerate-e2e` | Same functionality |

**Both execute the same agents and produce identical results.**

**Recommendation:**
- Use **root commands** (`/full-workflow`) for quick access
- Use **plugin commands** (`/test-gen:full-workflow`) for better organization and namespace isolation

See `PLUGIN_USAGE.md` for plugin installation and usage details.

