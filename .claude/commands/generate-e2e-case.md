---
description: Generate E2E test code and integrate into openshift-tests-private
argument-hint: [JIRA_KEY]
---

## Name
generate-e2e-case

## Synopsis
```
/generate-e2e-case JIRA_KEY
```

## Description
The `generate-e2e-case` command generates E2E test code based on test cases and integrates it into the openshift-tests-private repository.

This is the second step in the test generation workflow.

## Implementation
Executes the e2e_test_generation_openshift_private agent at `config/agents/e2e_test_generation_openshift_private.md`

The agent performs:
1. Repository setup and branch creation
2. Test code generation from test cases
3. Code integration and validation
4. Quality checks

Total execution time: ~120 seconds

## Return Value
- **Success**: E2E test code in `temp_repos/openshift-tests-private/`
- **Failure**: Error message indicating which step failed

## Examples

1. **Basic usage**:
   ```
   /generate-e2e-case HIVE-2883
   ```

2. **For different component**:
   ```
   /generate-e2e-case CCO-1234
   ```

## Arguments
- **$1** (required): JIRA issue key (e.g., HIVE-2883, CCO-1234)

## Prerequisites
- Test case must exist (run `/generate-test-case` first)
- GitHub CLI (`gh`) installed and authenticated
- Fork of openshift-tests-private configured

## Prerequisite Check
- Checks if test case exists in `test_artifacts/{COMPONENT}/{JIRA_KEY}/test_cases/`
- If missing, prompts to run `/generate-test-case` first

## Output Location
```
temp_repos/openshift-tests-private/
└── test/extended/{component}/
    └── {jira_key}_test.go
```

Branch: `ai-case-design-{JIRA_KEY}`

## Regenerate Mode
Use `/regenerate-e2e` to skip prerequisite checks and force regeneration.

## See Also
- `e2e_test_generation_openshift_private.md` - Agent implementation
- `/generate-test-case` - Previous step
- `/run-tests` - Next step: Execute tests
- `/regenerate-e2e` - Force regeneration

---

Execute E2E test generation agent for: **{args}**
