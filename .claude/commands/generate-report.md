# Generate Test Report

Execute the test report generation workflow for a JIRA issue.

## Usage
```
/generate-report HIVE-2883
/generate-report CCO-1234
```

## What this command does

1. **MANDATORY**: Read test_report_generation agent configuration
2. Load report template
3. Execute all 4 steps:
   - Step 1: Load Report Template (2s)
   - Step 2: Gather All Test Artifacts (10s parallel)
   - Step 3: Generate Comprehensive Report (30s)
   - Step 4: Report Validation (3s)
4. Generate comprehensive report in `test_artifacts/{COMPONENT}/{JIRA_KEY}/test_report.md`

## Arguments
- `$1` (required): JIRA issue key (e.g., HIVE-2883, CCO-1234)

## Execution Protocol

**CRITICAL**: Execute with maximum efficiency:
- ✅ Read `config/agents/test_report_generation.md`
- ✅ Execute EACH step in agent config manually
- ✅ Verify each step output before proceeding
- ✅ Use EXACT commands from config
- ✅ Handle missing artifacts gracefully
- ❌ NEVER use Task tool
- ❌ NEVER skip steps

## Prerequisites

This command requires existing test artifacts from previous workflow steps:
- test_requirements_output.md (from test_case_generation)
- test_coverage_matrix.md (from test_case_generation)
- E2E/Manual test cases (from test_case_generation)
- comprehensive_test_results.md (from test-executor, optional)

## Example
```
User: /generate-report HIVE-2883
→ Executes test_report_generation agent for HIVE-2883
→ Output: test_artifacts/hive/HIVE-2883/test_report.md
```

---

Execute test report generation agent for: **{args}**
