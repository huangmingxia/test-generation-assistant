---
description: Generate comprehensive test cases from JIRA issue
argument-hint: [JIRA_KEY]
---

## Name
generate-test

## Synopsis
```
/test-gen:generate-test JIRA_KEY
```

## Description
The `generate-test` command generates comprehensive test cases from a JIRA issue using systematic thinking framework.

Applies a 4-phase methodology:
1. **Requirements Analysis** - Extract and analyze JIRA requirements
2. **Test Strategy** - Define testing approach and coverage
3. **Test Case Design** - Create detailed test scenarios
4. **Validation** - Ensure completeness and quality

## Implementation
Executes the test case generation agent at `config/agents/test_case_generation.md`

The agent performs:
- Extracts requirements from JIRA via JIRA MCP
- Analyzes code changes using DeepWiki MCP (optional)
- Applies systematic thinking framework
- Generates test cases in Polarion-compatible format
- Outputs to: `test_artifacts/{JIRA_KEY}/test_cases/`

Execution time: ~90 seconds

## Return Value
- **Success**: Test cases generated in `test_artifacts/{JIRA_KEY}/`
- **Failure**: Error message with generation or validation errors

## Examples

1. **Basic usage**:
   ```
   /test-gen:generate-test HIVE-2883
   ```

2. **For Cloud Credential Operator**:
   ```
   /test-gen:generate-test CCO-1234
   ```

## Arguments
- **$1** (required): JIRA issue key (e.g., HIVE-2883, CCO-1234)

## Prerequisites
- JIRA MCP configured and accessible
- JIRA issue exists and is accessible
- DeepWiki MCP configured (optional, for code analysis)

## Output Structure
```
test_artifacts/{JIRA_KEY}/
├── phases/
│   ├── test_requirements_output.md    # Phase 1: Requirements
│   ├── test_strategy.md               # Phase 2: Strategy
│   └── test_case_design.md            # Phase 3: Design
├── test_cases/
│   └── {JIRA_KEY}_test_case.md        # Final test cases
└── test_coverage_matrix.md            # Coverage matrix
```

## Complete Workflow
1. `/test-gen:generate-test JIRA_KEY` ← This command
2. `/test-gen:generate-e2e JIRA_KEY`
3. `/test-gen:run-tests JIRA_KEY`
4. `/test-gen:submit-pr JIRA_KEY`

## See Also
- `test_case_generation.md` - Agent implementation
- `/test-gen:regenerate-test` - Regenerate without checks
- `/test-gen:generate-e2e` - Next step in workflow
- `/test-gen:full-workflow` - Automated complete workflow
