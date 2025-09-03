# 🧪 Test Execution Workflow

> **Execute generated E2E tests and analyze results**

## 🎯 Overview

This workflow executes previously generated E2E tests and analyzes the results. It requires the Test Case Generation Workflow to be completed first.

## 🏗️ Architecture

### 3-Phase Workflow Engine

1. **Environment Verification**  
   Use Task tool to call agent: `environment_checker` to verify test environment and prerequisites
2. **E2E Test Execution**  
   Use Task tool to call agent: `test-executor` to execute E2E tests in target environment
3. **Test Result Analysis**  
   Use Task tool to call agent: `test_result_analyzer` to analyze execution results and generate reports

**Execution Order**: Phase 1 → Phase 2 → Phase 3 (Sequential execution)

## 📊 Output Structure

```
workflow_outputs/{COMPONENT}/{JIRA_KEY}/test_execution_results/
├── environment_check_results.yaml      # Environment verification results
├── e2e_execution_log.txt              # E2E test execution logs
├── e2e_test_results.xml               # E2E test results in XML format
├── {JIRA_KEY}_execution_report.md     # Comprehensive execution report
└── {JIRA_KEY}_test_summary.yaml       # Test execution summary
```

## 🤖 Agent Architecture

### Core Execution Agents

| Agent | Purpose | Phase | Key Functions |
|-------|---------|-------|---------------|
| **environment_checker** | Verify test environment | 1 | Cluster connectivity, component status, resource validation |
| **test-executor** | Execute E2E tests | 2 | Test execution, progress monitoring, result collection |
| **test_result_analyzer** | Analyze results | 3 | Result parsing, failure analysis, report generation |

## ⚙️ Configuration

### Agent Configuration (`config/agents/`)

Each agent has its own configuration file:

```yaml
# config/agents/environment_checker.yaml
# config/agents/test-executor.yaml  
# config/agents/test_result_analyzer.yaml
```

### Workflow Configuration (`config/test_execution_workflow.yaml`)

Controls execution flow, phases, timeouts, and dependencies.

## 🔗 Integrations

### Cluster Integration
- **Purpose**: Connect to target OpenShift cluster
- **Requirements**: Active cluster connection, proper authentication
- **Validation**: Component deployment status, resource availability

### E2E Repository Integration
- **Purpose**: Access and execute generated E2E test code
- **Location**: Configured in component rules (`config/rules/{COMPONENT}-rules/component_rules_config.yaml`)
- **Access**: Read and execute permissions required

### Test Result Storage
- **Purpose**: Store and organize execution results
- **Location**: `workflow_outputs/{COMPONENT}/{JIRA_KEY}/test_execution_results/`
- **Format**: YAML, XML, Markdown, text logs

## 📋 Execution Checklist

### Pre-Execution Requirements
- [ ] Test Case Generation Workflow completed successfully
- [ ] Generated test artifacts available in `workflow_outputs/`
- [ ] Target cluster accessible and authenticated
- [ ] Component deployed and running in target environment

### Phase Completion

#### ✅ Phase 1 Complete
- [ ] Environment verification completed successfully
- [ ] `environment_check_results.yaml` generated
- [ ] All prerequisites validated

#### ✅ Phase 2 Complete
- [ ] E2E tests executed successfully
- [ ] Execution logs collected
- [ ] Test results captured

#### ✅ Phase 3 Complete
- [ ] Test results analyzed successfully
- [ ] Execution report generated
- [ ] Test summary created

## 🚀 Quick Start

### 1. Prerequisites
Ensure Test Case Generation Workflow has completed:
```bash
# Check generated artifacts exist
ls workflow_outputs/{COMPONENT}/{JIRA_KEY}/
```

### 2. Execute Workflow
```bash
# Set your parameters
JIRA_KEY="HIVE-2040"
COMPONENT="hive"

# Execute environment verification
# Use Task tool to call agent: environment_checker

# Execute E2E tests
# Use Task tool to call agent: test-executor

# Analyze results
# Use Task tool to call agent: test_result_analyzer
```

### 3. Review Results
Check outputs in `workflow_outputs/{COMPONENT}/{JIRA_KEY}/test_execution_results/`

## 📚 Related Documentation

- **[TEST_CASE_GENERATION_WORKFLOW.md](TEST_CASE_GENERATION_WORKFLOW.md)** - Generate test cases first
- **[CLAUDE.md](../CLAUDE.md)** - Claude Code instructions
- **[README.md](../README.md)** - Project overview and setup

## 🤝 Contributing

1. Follow existing execution patterns
2. Test with sample test cases
3. Update documentation for new features
4. Maintain backward compatibility

---

**Built for OpenShift E2E test execution**
