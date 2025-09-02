# 🧪 Test Execution Workflow Documentation

## 📋 Overview

The **Test Execution Workflow** is a dedicated workflow engine for executing tests using previously generated test cases. This workflow requires the Test Case Generation Workflow to be completed successfully before execution.

**Key Characteristics:**
- **Purpose**: Execute generated test cases and analyze results
- **Prerequisites**: Test Case Generation Workflow completion
- **Total Time**: 15 minutes (900 seconds)
- **Environment**: Requires active cluster connection and component deployment

## 🚀 Workflow Configuration

### Basic Information
- **Name**: `test_execution`
- **Version**: `1.0`
- **Description**: Dedicated workflow for test execution using generated test cases
- **Target Execution Time**: 15 minutes
- **Total Timeout**: 900 seconds

## 📥 Input Requirements

### Prerequisites
- **Required Workflow**: `test_case_generation` must be completed first
- **Required Inputs**:
  - `workflow_outputs/{COMPONENT}/{JIRA_KEY}/phases/test_requirements_output.yaml`
  - `workflow_outputs/{COMPONENT}/{JIRA_KEY}/phases/{JIRA_KEY}_test_strategy_output.yaml`
  - `workflow_outputs/{COMPONENT}/{JIRA_KEY}/test_cases/{JIRA_KEY}_test_case.md`
  - `workflow_outputs/{COMPONENT}/{JIRA_KEY}/phases/{JIRA_KEY}_e2e_test_results.md`

## 🔄 Workflow Phases

### Phase 1: Environment Verification (60s)
**Agent**: `environment_checker`
**Description**: Verify test environment and prerequisites
**Execution Order**: 1
**Dependencies**: None
**Timeout**: 60 seconds
**Critical Path**: Yes

#### Execution Steps
1. **Load Agent Config**
   - Action: `load_agent_config`
   - Config File: `config/agents/environment_checker.yaml`

2. **Validate Prerequisites**
   - Action: `validate_test_case_generation_outputs`
   - Required Files:
     - `test_requirements_output.yaml`
     - `{JIRA_KEY}_test_case.md`

3. **Check Environment**
   - Action: `execute_agent`
   - Tools: `cluster_connection`, `component_status`, `resource_availability`
   - Output Format: YAML

4. **Save Output**
   - Action: `save_to_file`
   - Path: `workflow_outputs/{COMPONENT}/{JIRA_KEY}/test_execution_results/environment_check_results.yaml`

### Phase 2: E2E Test Execution (480s)
**Agent**: `e2e_test_runner`
**Description**: Execute E2E tests in target environment
**Execution Order**: 2
**Dependencies**: Phase 1
**Timeout**: 480 seconds (8 minutes)
**Critical Path**: Yes

#### Execution Steps
1. **Wait for Dependencies**
   - Action: `wait_for_phase`
   - Wait For: Phase 1
   - Timeout: 10 seconds

2. **Load Agent Config**
   - Action: `load_agent_config`
   - Config File: `config/agents/e2e_test_runner.yaml`

3. **Load Test Inputs**
   - Action: `load_phase_outputs`
   - Inputs:
     - `{JIRA_KEY}_e2e_test_results.md`
     - `environment_check_results.yaml`

4. **Execute E2E Tests**
   - Action: `execute_agent`
   - Tools: `e2e_repository`, `test_runner`, `validation_checker`
   - Output Format: Mixed

5. **Save Execution Logs**
   - Action: `save_multiple_files`
   - Paths:
     - `e2e_execution_log.txt`
     - `e2e_test_results.xml`

### Phase 3: Manual Test Guidance (120s)
**Agent**: `test_case_generation`
**Description**: Provide manual test execution guidance
**Execution Order**: 3
**Dependencies**: Phase 1
**Parallel With**: Phase 2
**Timeout**: 120 seconds

#### Execution Steps
1. **Load Manual Test Cases**
   - Action: `load_phase_outputs`
   - Inputs: `{JIRA_KEY}_test_case.md`

2. **Prepare Manual Execution**
   - Action: `execute_agent`
   - Tools: `manual_test_preparation`, `environment_setup`
   - Output Format: Markdown

3. **Save Manual Guidance**
   - Action: `save_to_file`
   - Path: `manual_test_guidance.md`

### Phase 4: Test Result Analysis (180s)
**Agent**: `test_result_analyzer`
**Description**: Analyze test execution results and generate reports
**Execution Order**: 4
**Dependencies**: Phase 2, Phase 3
**Timeout**: 180 seconds

#### Execution Steps
1. **Wait for Dependencies**
   - Action: `wait_for_phase`
   - Wait For: Phase 2, Phase 3
   - Timeout: 15 seconds

2. **Load Agent Config**
   - Action: `load_agent_config`
   - Config File: `config/agents/test_result_analyzer.yaml`

3. **Load Execution Results**
   - Action: `load_phase_outputs`
   - Inputs:
     - `e2e_execution_log.txt`
     - `e2e_test_results.xml`
     - `manual_test_guidance.md`

4. **Analyze Results**
   - Action: `execute_agent`
   - Tools: `result_parser`, `failure_analyzer`, `recommendation_engine`
   - Output Format: Mixed

5. **Generate Final Report**
   - Action: `save_multiple_files`
   - Paths:
     - `{JIRA_KEY}_execution_report.md`
     - `{JIRA_KEY}_test_summary.yaml`

## ⚙️ Execution Strategy

### Parallel Execution
- **Enabled**: Yes
- **Max Concurrent Phases**: 2
- **Dependency Resolution**: Strict

### Environment Requirements
- **Cluster Required**: Yes
- **Resource Intensive**: Yes
- **Isolation Required**: Yes

### Failure Handling
- **Continue on Non-Critical Failure**: Yes
- **Critical Phases**: Phase 1, Phase 2
- **Retry Failed Tests**: Yes
- **Max Test Retries**: 2

## 🚀 Performance Configuration

### Resource Limits
- **Memory Limit**: 8GB
- **CPU Limit**: 16 cores
- **Disk Space**: 50GB
- **Priority Allocation**: Execution focused

### Timeouts
- **Individual Test Timeout**: 300 seconds (5 minutes)
- **Test Suite Timeout**: 1200 seconds (20 minutes)
- **Environment Setup Timeout**: 180 seconds (3 minutes)

### Error Handling
- **Retry on Failure**: Yes
- **Max Retries**: 3
- **Test Isolation**: Yes
- **Cleanup on Failure**: Yes

## 🔗 Integration Configuration

### Cluster Integration
- **Kubeconfig Path**: From global configuration
- **Connection Validation**: Yes
- **Resource Monitoring**: Yes

### Test Repository
- **E2E Repository Location**: From global configuration
- **Branch Management**: Yes
- **Test Execution Tracking**: Yes

### Reporting
- **JUnit XML Generation**: Yes
- **Log Aggregation**: Yes
- **Screenshot Capture**: Yes
- **Failure Diagnostics**: Yes

## 📁 Output Management

### Base Directory Structure
```
workflow_outputs/{COMPONENT}/{JIRA_KEY}/
├── test_execution_results/        # Main execution outputs
│   ├── environment_check_results.yaml
│   ├── e2e_execution_log.txt
│   ├── e2e_test_results.xml
│   ├── manual_test_guidance.md
│   ├── {JIRA_KEY}_execution_report.md
│   └── {JIRA_KEY}_test_summary.yaml
├── logs/                          # Execution logs
├── reports/                       # Generated reports
└── artifacts/                     # Test artifacts
```

### Artifact Collection
- **Test Logs**: Yes
- **Screenshots**: Yes
- **Cluster State Dumps**: Yes
- **Performance Metrics**: Yes

### Retention Policy
- **Execution Logs**: 30 days
- **Test Artifacts**: 7 days
- **Summary Reports**: 90 days

## 🧪 Test Categories

### E2E Tests
- **Enabled**: Yes
- **Priority**: High
- **Parallel Execution**: No

### Manual Tests
- **Enabled**: Yes
- **Priority**: Medium
- **Guidance Only**: Yes

### Integration Tests
- **Enabled**: No
- **Priority**: Low

### Performance Tests
- **Enabled**: No
- **Priority**: Low

## 🔍 Usage Instructions

### 1. Prerequisites Check
Ensure the Test Case Generation Workflow has completed successfully and generated all required outputs.

### 2. Environment Preparation
- Verify cluster connectivity
- Ensure component deployment is active
- Check resource availability

### 3. Execute Workflow
```bash
# The workflow will automatically:
# 1. Verify environment and prerequisites
# 2. Execute E2E tests (if available)
# 3. Provide manual test guidance
# 4. Analyze results and generate reports
```

### 4. Expected Outputs
After execution, you'll find outputs in:
```
workflow_outputs/{COMPONENT}/{JIRA_KEY}/test_execution_results/
├── environment_check_results.yaml
├── e2e_execution_log.txt
├── e2e_test_results.xml
├── manual_test_guidance.md
├── {JIRA_KEY}_execution_report.md
└── {JIRA_KEY}_test_summary.yaml
```

## 🚨 Troubleshooting

### Common Issues

1. **Environment Unavailable**
   - Check cluster connectivity
   - Verify component deployment status
   - Ensure resource availability

2. **Test Execution Failures**
   - Review E2E test configuration
   - Check cluster permissions
   - Verify test data availability

3. **Timeout Issues**
   - Monitor resource usage
   - Check network connectivity
   - Review test complexity

4. **Output Generation Failures**
   - Verify file permissions
   - Check disk space
   - Review agent configurations

### Performance Tuning

- **Memory**: Increase from 8GB if processing large test suites
- **CPU**: Scale up from 16 cores for faster parallel execution
- **Timeouts**: Adjust based on test complexity and network conditions

## 📋 Validation Checklist

Before using the Test Execution Workflow:

- [ ] Test Case Generation Workflow completed successfully
- [ ] All required input files exist
- [ ] Cluster connectivity verified
- [ ] Component deployment active
- [ ] Sufficient resources available
- [ ] Output directory writable
- [ ] Agent configurations loaded
- [ ] Dependencies resolved

## 🔄 Maintenance

### Regular Updates
1. **Monitor Performance**: Track execution times and resource usage
2. **Update Timeouts**: Adjust based on test complexity changes
3. **Review Configurations**: Ensure agent configs are up to date
4. **Cleanup Outputs**: Manage retention policies

### Version Compatibility
- Keep workflow compatible with Test Case Generation Workflow updates
- Test configuration changes with sample executions
- Maintain backward compatibility when possible

## 📚 Related Documentation

- [Test Case Generation Workflow](config/test_case_generation_workflow.yaml) - Prerequisite workflow
- [Workflow Guide](prompts/workflow_guide.md) - Complete workflow documentation
- [Execution Guide](prompts/execution_guide.md) - Execution best practices
- [Agent Configurations](config/agents/) - Individual agent settings
