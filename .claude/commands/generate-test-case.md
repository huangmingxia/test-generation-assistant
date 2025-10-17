---
description: Generate test cases from JIRA issue
argument-hint: [JIRA_KEY]
---

## Name
generate-test-case

## Synopsis
```
/generate-test-case JIRA_KEY
```

## Description
The `generate-test-case` command generates comprehensive test cases from a JIRA issue, including test requirements analysis, test strategy, and detailed test case design.

This is the first step in the test generation workflow.

## Implementation
Executes the test_case_generation agent at `config/agents/test_case_generation.md`

The agent performs 4 phases:
1. **Phase 1**: JIRA Requirements Analysis
2. **Phase 2**: Test Strategy Generation
3. **Phase 3**: Test Case Design
4. **Phase 4**: Test Coverage Matrix

Total execution time: ~90 seconds

## Return Value
- **Success**: Test artifacts in `test_artifacts/{COMPONENT}/{JIRA_KEY}/`
- **Failure**: Error message indicating which phase failed

## Examples

1. **Basic usage**:
   ```
   /generate-test-case HIVE-2883
   ```

2. **For different component**:
   ```
   /generate-test-case CCO-1234
   ```

## Arguments
- **$1** (required): JIRA issue key (e.g., HIVE-2883, CCO-1234)

## Prerequisites
- JIRA MCP configured and accessible
- Component rules exist in `config/rules/test_case_rules/`

## Output Structure
```
test_artifacts/{COMPONENT}/{JIRA_KEY}/
├── phases/
│   ├── test_requirements_output.md
│   └── test_strategy.md
├── test_cases/
│   └── {JIRA_KEY}_test_cases.md
└── test_coverage_matrix.md
```

## Execution Protocol
**CRITICAL**: Execute with maximum efficiency:
- ✅ Read `config/agents/test_case_generation.md`
- ✅ Execute EACH step in agent config manually
- ✅ Verify each step output before proceeding
- ✅ Use EXACT commands from config
- ❌ NEVER skip steps

## Regenerate Mode
Use `/regenerate-test` to skip prerequisite checks and force regeneration.

## See Also
- `test_case_generation.md` - Agent implementation
- `/generate-e2e-case` - Next step: E2E code generation
- `/regenerate-test` - Force regeneration

---

Execute test case generation agent for: **{args}**
