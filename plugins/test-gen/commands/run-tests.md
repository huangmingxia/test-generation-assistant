---
description: Execute E2E tests and generate comprehensive test reports
argument-hint: [JIRA_KEY]
---

## Name
run-tests

## Synopsis
```
/test-gen:run-tests JIRA_KEY
```

## Description
The `run-tests` command executes E2E tests for the specified JIRA issue and generates comprehensive test execution reports.

The command locates the generated E2E test code, executes it against the configured OpenShift cluster, and produces detailed test reports including pass/fail status, execution logs, and error analysis.

## Implementation
Executes the test executor agent at `config/agents/test-executor.md`

The agent performs:
- Locates E2E test code in openshift-tests-private fork
- Configures test environment using provided kubeconfig
- Executes tests using extended-platform-tests binary
- Captures test output and logs
- Generates comprehensive test report
- Analyzes failures and provides debugging information

Execution time: ~5-10 minutes (depends on test complexity)

## Return Value
- **Success**: Test execution report generated in `test_artifacts/{JIRA_KEY}/test_report.md`
- **Failure**: Error message with execution or environment configuration errors

## Examples

1. **Basic usage**:
   ```
   /test-gen:run-tests HIVE-2883
   ```

2. **After E2E generation**:
   ```
   /test-gen:generate-e2e HIVE-2883
   /test-gen:run-tests HIVE-2883
   ```

3. **Complete workflow**:
   ```
   /test-gen:generate-test HIVE-2883
   /test-gen:generate-e2e HIVE-2883
   /test-gen:run-tests HIVE-2883
   ```

## Arguments
- **$1** (required): JIRA issue key (e.g., HIVE-2883, CCO-1234)

## Prerequisites
- E2E test code exists (run `/test-gen:generate-e2e` first)
- KUBECONFIG environment variable set to valid OpenShift cluster config
- OpenShift cluster accessible and operational
- extended-platform-tests binary built (or compiled from source)
- Test environment matches expected platform (AWS, Azure, GCP, etc.)

## Output Structure
```
test_artifacts/{JIRA_KEY}/
├── test_report.md               # Comprehensive test report
├── test_execution_log.txt       # Raw test execution output
└── test_results.json            # Structured test results
```

## Complete Workflow
1. `/test-gen:generate-test JIRA_KEY`
2. `/test-gen:generate-e2e JIRA_KEY`
3. `/test-gen:run-tests JIRA_KEY` ← This command
4. `/test-gen:submit-pr JIRA_KEY`

## See Also
- `test-executor.md` - Agent implementation
- `/test-gen:generate-e2e` - Previous step (required)
- `/test-gen:generate-report` - Generate additional reports
- `/test-gen:submit-pr` - Next step in workflow
- `/test-gen:full-workflow` - Automated complete workflow
