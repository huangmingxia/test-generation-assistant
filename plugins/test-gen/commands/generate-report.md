---
description: Generate comprehensive test execution and coverage report
argument-hint: [JIRA_KEY]
---

## Name
generate-report

## Synopsis
```
/test-gen:generate-report JIRA_KEY
```

## Description
Generates comprehensive test report including execution results, coverage analysis, and recommendations.

## Implementation
Executes test_report_generation agent at `config/agents/test_report_generation.md`

Execution time: ~30 seconds

## Examples
1. **Generate report after tests**:
   ```
   /test-gen:run-tests HIVE-2883
   /test-gen:generate-report HIVE-2883
   ```

## Arguments
- **$1** (required): JIRA issue key

## Prerequisites
- Test artifacts exist

## Output
```
test_artifacts/{JIRA_KEY}/test_report.md
```

## See Also
- `/test-gen:run-tests` - Execute tests first
- `/test-gen:submit-pr` - Include report in PR
