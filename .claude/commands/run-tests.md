---
description: Execute E2E tests and generate comprehensive test reports
argument-hint: [JIRA_KEY]
---

## Name
run-tests

## Synopsis
```
/run-tests JIRA_KEY
```

## Description
The `run-tests` command executes E2E tests for a JIRA issue and generates comprehensive test execution reports.

This is the third step in the test generation workflow.

## Implementation
Executes the test-executor agent at `config/agents/test-executor.md`

The agent performs:
1. Locate E2E test code
2. Execute tests in appropriate environment
3. Collect test results
4. Generate test execution report

Total execution time: ~180 seconds

## Return Value
- **Success**: Test execution report in `test_artifacts/{COMPONENT}/{JIRA_KEY}/phases/`
- **Failure**: Error message with test failure details

## Examples

1. **Basic usage**:
   ```
   /run-tests HIVE-2883
   ```

2. **For different component**:
   ```
   /run-tests CCO-1234
   ```

## Arguments
- **$1** (required): JIRA issue key (e.g., HIVE-2883, CCO-1234)

## Prerequisites
- E2E test code must exist (run `/generate-e2e-case` first)
- Test environment must be configured
- OpenShift cluster kubeconfig accessible

## Output Structure
```
test_artifacts/{COMPONENT}/{JIRA_KEY}/phases/
└── comprehensive_test_results.md
```

## Test Execution Details
- Tests run against OpenShift cluster
- Results include:
  - Test pass/fail status
  - Execution logs
  - Error diagnostics (if failed)
  - Performance metrics

## See Also
- `test-executor.md` - Agent implementation
- `/generate-e2e-case` - Previous step
- `/generate-report` - Generate comprehensive report
- `/submit-pr` - Next step: Submit PR

---

Execute test-executor agent for: **{args}**
