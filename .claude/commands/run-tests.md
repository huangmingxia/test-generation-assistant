# Run E2E Tests

Execute E2E tests and generate comprehensive test reports.

## Usage
```
/run-tests HIVE-2883
/run-tests CCO-1234
```

## What this command does

1. Load test-executor agent configuration
2. Locate E2E test code for the JIRA issue
3. Execute tests in appropriate environment
4. Generate test execution report
5. Analyze results and provide summary

## Arguments
- `$1` (required): JIRA issue key (e.g., HIVE-2883)

## Prerequisites
- E2E test code must exist (run `/generate-e2e` first)
- Test environment must be configured (kubeconfig accessible)

## Example
```
User: /run-tests HIVE-2883
→ Executes test-executor agent
→ Runs E2E tests
→ Output: Test report in workflow_outputs/hive/HIVE-2883/phases/
```

---

Execute test-executor agent for: **{args}**
