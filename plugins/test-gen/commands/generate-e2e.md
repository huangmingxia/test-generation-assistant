---
description: Generate E2E test code for openshift-tests-private repository
argument-hint: [JIRA_KEY]
---

## Name
generate-e2e

## Synopsis
```
/test-gen:generate-e2e JIRA_KEY
```

## Description
The `generate-e2e` command generates executable Ginkgo-based E2E test code for the openshift-tests-private repository based on test case specifications.

The command integrates the generated E2E code directly into your fork of openshift-tests-private and creates a feature branch for PR submission.

## Implementation
Executes the E2E test generation agent at `config/agents/e2e_test_generation_openshift_private.md`

The agent performs:
- Reads test case from: `test_artifacts/{JIRA_KEY}/test_cases/`
- Analyzes repository patterns in openshift-tests-private
- Generates Ginkgo E2E test code following repository conventions
- Creates feature branch: `ai-case-design-{JIRA_KEY}`
- Commits E2E code to the branch

Execution time: ~2-3 minutes

## Return Value
- **Success**: E2E test code generated and committed to feature branch
- **Failure**: Error message with generation or validation errors

## Examples

1. **Basic usage**:
   ```
   /test-gen:generate-e2e HIVE-2883
   ```

2. **After test case generation**:
   ```
   /test-gen:generate-test HIVE-2883
   /test-gen:generate-e2e HIVE-2883
   ```

## Arguments
- **$1** (required): JIRA issue key (e.g., HIVE-2883, CCO-1234)

## Prerequisites
- Test case file exists in `test_artifacts/{JIRA_KEY}/test_cases/`
- Fork of openshift-tests-private configured in environment
- GitHub credentials configured (for pushing to fork)
- `/test-gen:generate-test` has been run first

## Output Structure
```
~/auto-test-case/openshift-tests-private/
└── test/extended/cluster_operator/{component}/
    └── {jira_key}_test.go        # Generated E2E test

test_artifacts/{JIRA_KEY}/
└── e2e_test_info.json            # Metadata about generated E2E test
```

## Complete Workflow
1. `/test-gen:generate-test JIRA_KEY`
2. `/test-gen:generate-e2e JIRA_KEY` ← This command
3. `/test-gen:run-tests JIRA_KEY`
4. `/test-gen:submit-pr JIRA_KEY`

## See Also
- `e2e_test_generation_openshift_private.md` - Agent implementation
- `/test-gen:generate-test` - Previous step (required)
- `/test-gen:regenerate-e2e` - Regenerate without prerequisite checks
- `/test-gen:run-tests` - Next step in workflow
- `/test-gen:full-workflow` - Automated complete workflow
