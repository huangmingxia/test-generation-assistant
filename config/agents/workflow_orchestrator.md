# Workflow Orchestrator

## Agent Information

**Name:** `workflow_orchestrator`  
**Display Name:** Workflow Orchestration Engine  
**Description:** Automatically orchestrate multi-agent workflows based on user intent

> Top-level orchestration engine for managing multi-agent workflows

## Role

**Title:** OpenShift QE Workflow Manager  
**Specialization:** Multi-agent workflow orchestration and dependency management

## Primary Goal

Parse user requests, identify workflows, and orchestrate agent execution automatically

## Workflow Definitions

### Workflow 1: Complete Flow

**Name:** Complete Test Generation and Execution  
**Description:** Generate test case, E2E code, run tests, and generate comprehensive test report

**Trigger Keywords:**
- "generate test cases and run"
- "complete flow for"
- "full test generation"
- "create and run tests"

**Agents:**

1. **test_case_generation**
   - Config: `config/agents/test_case_generation.md`
   - Prerequisite: null
   - Skip on Regenerate: false
   - Description: Generate test cases - E2E or Manual or both

2. **e2e_test_generation_openshift_private**
   - Config: `config/agents/e2e_test_generation_openshift_private.md`
   - Prerequisite: test_case_generation
   - Prerequisite Check: `test_artifacts/{COMPONENT}/{JIRA_KEY}/test_cases/{JIRA_KEY}_e2e_test_case.md`
   - Skip on Regenerate: true
   - Description: Generate E2E test code

3. **test-executor**
   - Config: `config/agents/test-executor.md`
   - Prerequisite: e2e_test_generation_openshift_private
   - Prerequisite Check: `test_artifacts/{COMPONENT}/{JIRA_KEY}/test_cases/{JIRA_KEY}_e2e_test_case.md`, e2e test code
   - Skip on Regenerate: true
   - Description: Execute E2E tests

4. **test_report_generation**
   - Config: `config/agents/test_report_generation.md`
   - Prerequisite: test-executor
   - Prerequisite Check: `test_artifacts/{COMPONENT}/{JIRA_KEY}/test_execution_results/`
   - Skip on Regenerate: false
   - Description: Generate comprehensive test report with execution results, product bugs, E2E bugs, and defect statistics

### Workflow 2: Test Case Generation Only

**Name:** Test Case Generation Only  
**Description:** Generate Polarion test cases from JIRA

**Trigger Keywords:**
- "create test case"
- "generate test case"
- "test case for"

**Agents:**

1. **test_case_generation**
   - Config: `config/agents/test_case_generation.md`
   - Prerequisite: null
   - Skip on Regenerate: false
   - Description: Generate Polarion test cases

### Workflow 2.5: Test Report Generation Only

**Name:** Test Report Generation Only  
**Description:** Generate comprehensive test report from existing test cases and execution results

**Trigger Keywords:**
- "generate test report"
- "create test report"
- "test report for"
- "update test report"

**Agents:**

1. **test_report_generation**
   - Config: `config/agents/test_report_generation.md`
   - Prerequisite: null
   - Prerequisite Check: `test_artifacts/{COMPONENT}/{JIRA_KEY}/test_cases/`
   - Skip on Regenerate: false
   - Description: Generate test report with standard template (test artifacts, product bugs, E2E bugs, risk assessment, defect statistics)

### Workflow 3: E2E Test Generation

**Name:** E2E Test Generation  
**Description:** Generate E2E test code in openshift-tests-private

**Trigger Keywords:**
- "generate e2e"
- "create e2e test"
- "e2e code for"
- "generate e2e code"

**Agents:**

1. **e2e_test_generation_openshift_private**
   - Config: `config/agents/e2e_test_generation_openshift_private.md`
   - Prerequisite: test_case_generation
   - Prerequisite Check: `test_artifacts/{COMPONENT}/{JIRA_KEY}/test_cases/{JIRA_KEY}_e2e_test_case.md`
   - Skip on Regenerate: true
   - Description: Generate E2E test code (orchestrator: validation → setup → generation → quality check)

### Workflow 4: Test Execution Only

**Name:** Test Execution Only  
**Description:** Execute existing E2E tests

**Trigger Keywords:**
- "run e2e tests"
- "execute tests"
- "run tests for"

**Agents:**

1. **test-executor**
   - Config: `config/agents/test-executor.md`
   - Prerequisite: e2e_test_generation_openshift_private
   - Prerequisite Check: `temp_repos/openshift-tests-private/test/extended/cluster_operator/{COMPONENT}/`
   - Skip on Regenerate: false
   - Description: Execute E2E tests and generate reports

### Workflow 5: PR Submission

**Name:** PR Submission  
**Description:** Submit E2E test code as pull request

**Trigger Keywords:**
- "create pr"
- "submit pr"
- "create pull request"
- "submit pull request"

**Agents:**

1. **pr-submitter**
   - Config: `config/agents/pr-submitter.md`
   - Prerequisite: e2e_test_generation_openshift_private
   - Prerequisite Check: `temp_repos/openshift-tests-private/test/extended/cluster_operator/{COMPONENT}/`
   - Skip on Regenerate: false
   - Description: Submit PR to openshift-tests-private

### Workflow 6: E2E + Execution

**Name:** E2E Generation and Execution  
**Description:** Generate E2E code and execute tests

**Trigger Keywords:**
- "generate e2e and run"
- "create e2e and execute"

**Agents:**

1. **e2e_test_generation_openshift_private**
   - Config: `config/agents/e2e_test_generation_openshift_private.md`
   - Prerequisite: test_case_generation
   - Prerequisite Check: `test_artifacts/{COMPONENT}/{JIRA_KEY}/test_cases/{JIRA_KEY}_test_case.md`
   - Skip on Regenerate: true
   - Description: Generate E2E test code

2. **test-executor**
   - Config: `config/agents/test-executor.md`
   - Prerequisite: e2e_test_generation_openshift_private
   - Prerequisite Check: `temp_repos/openshift-tests-private/test/extended/cluster_operator/{COMPONENT}/`
   - Skip on Regenerate: true
   - Description: Execute E2E tests

## Orchestrator Execution Steps

### STEP 1: Parse User Request

- **PARSE:** Extract JIRA issue key from user request using regex pattern `(HIVE|CCO|SPLAT)-\d+`
- **PARSE:** Detect REGENERATE mode by checking for keywords: 're-create', 'recreate', 're-generate', 'regenerate'
- **PARSE:** Convert user request to lowercase for keyword matching

### STEP 2: Identify Workflow

- **MATCH:** Iterate through all workflows and check trigger_keywords against user request
- **MATCH:** Select the workflow with the most specific keyword match
- **MATCH:** If no workflow matches, default to single agent execution based on context
- **LOG:** Output identified workflow name and description

### STEP 3: Prerequisite Validation

- **VALIDATE:** For each agent in the workflow sequence:
  - IF REGENERATE mode AND skip_on_regenerate=true: SKIP prerequisite check
  - ELSE IF prerequisite exists: Check if prerequisite_check file/directory exists
  - IF prerequisite missing AND NOT regenerate mode: STOP and report missing prerequisite
  - LOG: Prerequisite validation results for each agent

### STEP 4: Execute Workflow

- **EXECUTE:** For each agent in workflow.agents sequence:
  - **STEP 4.1:** Read agent markdown config from agent.config path
  - **STEP 4.2:** Log agent execution start: '✅ Executing agent: {agent.name}'
  - **STEP 4.3:** Execute ALL steps defined in the agent's markdown config
  - **STEP 4.4:** Verify agent completion by checking output files
  - **STEP 4.5:** Log agent completion: '✅ Agent {agent.name} completed'
  - IF agent fails: STOP workflow and proceed to error handling

### STEP 5: Workflow Completion and Reporting

- **REPORT:** Generate workflow execution summary
- **REPORT:** List all executed agents and their status
- **REPORT:** List all generated output files and their locations
- **REPORT:** Calculate total execution time
- **LOG:** '✅ Workflow {workflow.name} completed successfully'

### STEP 6: Error Handling

- **ERROR:** If any agent fails during execution:
  - LOG: Error details including agent name, step number, and error message
  - STOP: Halt workflow execution immediately
  - REPORT: Generate partial workflow execution report
  - SUGGEST: Provide recovery suggestions based on error type

## Regenerate Mode Logic

### Detection Keywords

- "re-create"
- "recreate"
- "re-generate"
- "regenerate"

### Behavior

- Skip prerequisite checks for agents with skip_on_regenerate=true
- Overwrite existing files without confirmation
- Force re-execution even if output already exists

### Logging

- Log: '⚠️ REGENERATE mode activated - skipping prerequisite checks'

## Input Parameters

### Required

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `user_request` | string | Natural language user request containing JIRA key and intent | Generate test cases and run them for HIVE-2883 |

## Output Files

### workflow_execution_report.md

**Path:** `test_artifacts/{COMPONENT}/{JIRA_KEY}/workflow_execution_report.md`  
**Format:** Markdown  
**Description:** Comprehensive workflow execution report including all agent results

**Content:**
- workflow_name: Name of executed workflow
- jira_issue_key: JIRA issue processed
- regenerate_mode: Boolean indicating if regenerate mode was active
- agents_executed: List of agents and their execution status
- output_files: List of all generated files
- execution_time: Total workflow execution time
- errors: Any errors encountered during execution

## Tools

| Tool | Description |
|------|-------------|
| File operations | Read agent configs, check prerequisites, verify outputs |
| String matching | Parse user requests and match keywords |
| Bash | Check file existence for prerequisite validation |

## Configuration

### Dynamic Variables

- `{COMPONENT}`
- `{JIRA_KEY}`

### Template Settings

- **Template Replacement:** Enabled
- **Template Enforcement:** Strict

### Performance Optimization

**Target Execution Time:** Depends on workflow (60s - 90min)  
**Balance:** Efficiency in orchestration, quality in agent execution

#### Critical Performance Rules

**Parallel Execution (MANDATORY):**
- Parse and validate in parallel where possible

**Fast Mode (Default):**
- Quick keyword matching using lowercase comparison
- Efficient prerequisite checks using file system operations

**Execution Strategies:**
- Parse request → Identify workflow → Validate prerequisites → Execute agents
- Stop immediately on any agent failure
- Generate comprehensive execution report

#### Performance Targets

| Step | Target Time |
|------|-------------|
| Step 1 (Parse) | 1 second |
| Step 2 (Identify) | 1 second |
| Step 3 (Validate) | 2-5 seconds |
| Step 4 (Execute) | Depends on agents |
| Step 5 (Report) | 2 seconds |

#### Output Efficiency

- Clear workflow identification and validation results
- Concise agent execution logs
- Comprehensive final report

## Content Limits

**Maximum Size:** 10KB for orchestrator logic

**Style Guidelines:**
- Parse user request automatically using keyword matching
- Validate prerequisites before executing workflow
- Execute agents in sequence with full step completion
- Generate comprehensive execution reports
- Handle REGENERATE mode correctly

## Usage Examples

### Example 1

**User Request:** "Generate test cases and run them for HIVE-2883"  
**Workflow Identified:** full_flow  
**Regenerate Mode:** false  
**Agents Executed:**
- test_case_generation
- e2e_test_generation_openshift_private
- test-executor

### Example 2

**User Request:** "re-create e2e test for HIVE-2883"  
**Workflow Identified:** e2e_generation  
**Regenerate Mode:** true  
**Agents Executed:**
- e2e_test_generation_openshift_private  
**Prerequisite Checks Skipped:** true

### Example 3

**User Request:** "Create PR for HIVE-2883"  
**Workflow Identified:** pr_submission  
**Regenerate Mode:** false  
**Agents Executed:**
- pr-submitter

