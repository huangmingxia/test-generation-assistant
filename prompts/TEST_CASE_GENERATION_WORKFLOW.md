# 🧪 Test Case Generation Workflow

> **AI-powered automated test case generation system**

## 🎯 Overview

This workflow automatically generates comprehensive test cases from JIRA tickets using a 4-phase AI engine. It supports OpenShift Hive (`HIVE-*`) and Cloud Credential Operator (`CCO-*`) components, producing   Markdown test case outputs.

## 🏗️ Architecture

### 4-Phase Workflow Engine

1. **Requirements Gathering**  
   Use Task tool to call agent: `requirements_gathering` to extract test requirements from JIRA tickets  
2. **Test Strategy Analysis**  
   Use Task tool to call agent: `test_strategy_analysis` to determine optimal test execution strategy
3. **Test Case Generation**  
   Use Task tool to call agent: `test_case_generation` to aggregate checkpoints and generate executable test scenarios
4. **E2E Test Code Generation**  
   Use Task tool to call agent: `e2e_test_generation` to generate actual E2E test code (.go files) and integrate them into the configured E2E repository (repository location dynamically loaded from `config/rules/{COMPONENT}-rules/component_rules_config.yaml`)

**Execution Order**: Phase 1 → Phase 2 → Phase 3 → Phase 4 (Sequential with some parallel execution possible)


## 📊 Output Structure

```
workflow_outputs/{COMPONENT}/{JIRA_KEY}/
├── phases/                        # Phase outputs
│   ├── test_requirements_output.yaml
│   ├── {JIRA_KEY}_test_strategy_output.yaml
│   └── {JIRA_KEY}_e2e_test_results.md    # ONLY generated if e2e_decision.required == YES
└── test_cases/                    # Generated test cases
    └── {JIRA_KEY}_test_case.md    # Markdown format
```

## 🚨 CRITICAL: Phase 4 Conditional Execution

**MANDATORY RULE**: Phase 4 (`e2e_test_generation`) is **CONDITIONALLY EXECUTED** based on Phase 2 decision:

- ✅ **IF** `e2e_decision.required == YES` → Execute Phase 4 and generate E2E test code
- ❌ **IF** `e2e_decision.required == NO` → **SKIP Phase 4 COMPLETELY** (no files generated)

**Configuration Rule**: `execute_only_if: "e2e_decision.required == YES"` in `config/test_case_generation_workflow.yaml:99`

## 🤖 Agent Architecture

### Core Workflow Agents

| Agent | Purpose | Phase | Key Functions |
|-------|---------|-------|---------------|
| **requirements_gathering** | Extract JIRA requirements | 1 | JIRA data extraction, requirement parsing |
| **test_strategy_analysis** | Determine test strategy | 2 | Rule application, strategy decision |
| **test_case_generation** | Generate test scenarios | 3 | Template processing, test case creation |
| **e2e_test_generation** | Generate actual E2E test code (.go files) | 4 | Go code generation, repository integration, compilation validation |

## 🔗 Integrations

### JIRA Integration
- **MCP Endpoint**: `jira-mcp-snowflake`
- **Data Source**: Snowflake-backed JIRA data warehouse
- **Timeout**: 30 seconds per query
- **Purpose**: Extract ticket requirements and details

### DeepWiki Integration
- **MCP Endpoint**: `DeepWiki MCP`
- **Purpose**: Code change analysis and testing implications
- **Parallel Queries**: Up to 8 concurrent connections
- **Timeout**: 30 seconds per query

### GitHub Integration
- **Dynamic Repository Selection**: Repository determined by component configuration
  - Hive: Uses `openshift-tests-private` (shared test repository)
  - CCO: Could use component-specific repo or shared repo (configurable)
- **Repository Management**: Automatic branch creation (`ai-case-design-{JIRA_KEY}`)
- **File Operations**: Direct file creation in dynamically selected E2E repositories
- **Version Control**: Git integration for test code management

## 📋 Execution Checklist

### Pre-Execution Requirements
- [ ] JIRA issue exists and is accessible
- [ ] E2E repository paths are accessible and writable
- [ ] Component configuration loaded from `config/components.yaml`
- [ ] Dynamic variables initialized (`{COMPONENT}`, `{JIRA_KEY}`)

### Phase Completion Markers

#### ✅ Phase 1 Complete
- [ ] JIRA issue data retrieved successfully
- [ ] `test_requirements_output.yaml` generated

#### ✅ Phase 2 Complete
- [ ] Phase 1 output loaded successfully
- [ ] Component rules applied
- [ ] `test_strategy_output.yaml` generated
- [ ] E2E decision clear (YES/NO)
- [ ] Execution time ≤ 60 seconds

#### ✅ Phase 3&4 Parallel Complete
- [ ] Phase 3 always executed: Test cases generated
- [ ] Phase 4 conditionally executed: E2E generation **ONLY IF** `e2e_decision.required == YES`
- [ ] If Phase 4 skipped: Only Phase 3 completes within 150 seconds
- [ ] If Phase 4 executed: Both complete within 150 seconds
- [ ] No inter-phase waiting time

## 🚀 Performance Optimization

### Execution Modes

**Standard Mode** (4 minutes):
- Phase 1: 60s
- Phase 2: 60s
- Phase 3: 150s (parallel)
- Phase 4: 150s (parallel)

## 🛠️ Advanced Features

### Component-Specific Rules
- **Hive Rules**: E2E mandatory steps, test case guidelines, operator testing patterns
- **CCO Rules**: Credential management testing, cloud provider integration
- **Common Rules**: Universal E2E pattern learning, shared testing principles

### E2E Code Generation (Phase 4)
- **Repository Selection**: Dynamically determined from `config/rules/{COMPONENT}-rules/component_rules_config.yaml`
  - `e2e_type: "test_private"` → Uses shared test repository (e.g., openshift-tests-private)
  - `e2e_type: "component_repo"` → Uses component-specific repository (e.g., hive repo)
- **Automatic Branch Creation**: `ai-case-design-{JIRA_KEY}`
- **Actual Go Code Generation**: Creates working .go test files in the configured E2E repository
- **Code Integration**: Direct file updates and compilation in target E2E repository
- **Syntax Validation**: Repository-specific validation (`make all` for test_private, `make verify` for component_repo)
- **Test Data Creation**: YAML templates and configuration files in appropriate test data directories
- **AI Attribution**: Adds AI authorship headers to generated code

### Error Handling and Fallbacks
- **Component Detection**: Auto-detection from JIRA key prefix
- **Missing Configuration**: Fallback to default rules and templates
- **Fast-Fail**: Immediate stop on critical path failures
- **Retry Logic**: Automatic retry for transient failures

## 📝 Adding New Components

1. **Update** `config/components.yaml` with component definition
2. **Create** component-specific rules in `config/rules/{COMPONENT}-rules/`
3. **Add** examples in `config/examples/{COMPONENT}-examples/`
4. **Test** with sample JIRA ticket

## 📚 Related Documentation

- **[User Configuration Guide](USER_CONFIGURATION_GUIDE.md)** - Complete setup for new components
- **[Checklist](STARTUP_CHECKLIST.md)** - Comprehensive execution checklist
- **[CLAUDE.md](../CLAUDE.md)** - Project instructions for Claude Code

## 🤝 Contributing

1. Follow existing configuration patterns
2. Test with sample JIRA tickets
3. Update documentation for new features
4. Maintain backward compatibility

---