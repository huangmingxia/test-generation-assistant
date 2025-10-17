---
description: Regenerate E2E test code without prerequisite checks (force regeneration)
argument-hint: [JIRA_KEY]
---

## Name
regenerate-e2e

## Synopsis
```
/test-gen:regenerate-e2e JIRA_KEY
```

## Description
Regenerates E2E test code, skipping prerequisite checks. Use when updating existing E2E tests.

## Implementation
Executes e2e_test_generation_openshift_private agent at `config/agents/e2e_test_generation_openshift_private.md` with regenerate mode.

Execution time: ~2-3 minutes

## Examples
1. **Regenerate after test case update**:
   ```
   /test-gen:regenerate-e2e HIVE-2883
   ```

## Arguments
- **$1** (required): JIRA issue key

## Use Cases
- Test cases were updated
- E2E code needs improvement

## See Also
- `/test-gen:generate-e2e` - Normal generation
- `/test-gen:regenerate-test` - Regenerate test cases
