---
description: Execute complete test generation workflow (test case → E2E code → run tests)
argument-hint: [JIRA_KEY]
---

## Name
full-workflow

## Synopsis
```
/test-gen:full-workflow JIRA_KEY
```

## Description
The `full-workflow` command executes the complete test generation workflow, automating all steps from test case generation to test execution.

This command combines three agents in sequence:
1. Test case generation
2. E2E test code generation
3. Test execution and reporting

## Implementation
Executes the workflow orchestrator agent at `config/agents/workflow_orchestrator.md`

The orchestrator performs:
1. **test_case_generation** - Generates comprehensive test cases from JIRA
2. **e2e_test_generation_openshift_private** - Creates E2E test code
3. **test-executor** - Runs tests and generates report

Total execution time: ~3-4 minutes

## Return Value
- **Success**: Complete workflow outputs in `test_artifacts/{JIRA_KEY}/`
- **Failure**: Error message indicating which phase failed

## Examples

1. **Basic usage**:
   ```
   /test-gen:full-workflow HIVE-2883
   ```

2. **For different component**:
   ```
   /test-gen:full-workflow CCO-1234
   ```

## Arguments
- **$1** (required): JIRA issue key (e.g., HIVE-2883, CCO-1234)

## Prerequisites
- JIRA MCP configured and accessible
- GitHub CLI (`gh`) installed and authenticated
- OpenShift cluster kubeconfig available (for test execution)
- Fork of openshift-tests-private configured

## Output Structure
```
test_artifacts/{JIRA_KEY}/
├── phases/
│   ├── test_requirements_output.md
│   ├── test_strategy.md
│   └── test_case_design.md
├── test_cases/
│   └── {JIRA_KEY}_test_case.md
├── test_report.md
└── e2e_test_info.json
```

## Complete Workflow
This IS the complete workflow. For step-by-step control:
1. `/test-gen:generate-test JIRA_KEY`
2. `/test-gen:generate-e2e JIRA_KEY`
3. `/test-gen:run-tests JIRA_KEY`
4. `/test-gen:submit-pr JIRA_KEY`

## See Also
- `workflow_orchestrator.md` - Orchestrator implementation
- `/test-gen:generate-test` - Manual test case generation
- `/test-gen:generate-e2e` - Manual E2E generation
- `/test-gen:run-tests` - Manual test execution
