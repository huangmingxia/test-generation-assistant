---
description: Generate comprehensive test report for a JIRA issue
argument-hint: [JIRA_KEY]
---

## Name
generate-report

## Synopsis
```
/generate-report JIRA_KEY
```

## Description
The `generate-report` command generates a comprehensive test report that consolidates all test artifacts including test cases, coverage matrix, and execution results.

This is an optional step that can be executed after test execution.

## Implementation
Executes the test_report_generation agent at `config/agents/test_report_generation.md`

The agent performs 4 steps:
1. **Step 1**: Load Report Template
2. **Step 2**: Gather All Test Artifacts
3. **Step 3**: Generate Comprehensive Report
4. **Step 4**: Report Validation

Total execution time: ~45 seconds

## Return Value
- **Success**: Test report in `test_artifacts/{COMPONENT}/{JIRA_KEY}/test_report.md`
- **Failure**: Error message indicating missing artifacts

## Examples

1. **Basic usage**:
   ```
   /generate-report HIVE-2883
   ```

2. **For different component**:
   ```
   /generate-report CCO-1234
   ```

## Arguments
- **$1** (required): JIRA issue key (e.g., HIVE-2883, CCO-1234)

## Prerequisites
This command requires existing test artifacts from previous workflow steps:
- test_requirements_output.md (from test_case_generation)
- test_coverage_matrix.md (from test_case_generation)
- E2E/Manual test cases (from test_case_generation)
- comprehensive_test_results.md (from test-executor, optional)

## Output Structure
```
test_artifacts/{COMPONENT}/{JIRA_KEY}/
└── test_report.md
```

## Report Contents
The generated report includes:
- JIRA issue summary
- Test requirements
- Test strategy
- Test cases (E2E and Manual)
- Test coverage matrix
- Test execution results (if available)

## Execution Protocol
**CRITICAL**: Execute with maximum efficiency:
- ✅ Read `config/agents/test_report_generation.md`
- ✅ Execute EACH step in agent config manually
- ✅ Verify each step output before proceeding
- ✅ Handle missing artifacts gracefully
- ❌ NEVER skip steps

## See Also
- `test_report_generation.md` - Agent implementation
- `/run-tests` - Previous step
- Report template: `config/templates/test_report_template.md`

---

Execute test report generation agent for: **{args}**
