---
description: Regenerate test cases without prerequisite checks (force regeneration)
argument-hint: [JIRA_KEY]
---

## Name
regenerate-test

## Synopsis
```
/test-gen:regenerate-test JIRA_KEY
```

## Description
Regenerates test cases, skipping prerequisite checks. Use when updating existing test cases.

## Implementation
Executes test_case_generation agent at `config/agents/test_case_generation.md` with regenerate mode.

Execution time: ~90 seconds

## Examples
1. **Regenerate after JIRA update**:
   ```
   /test-gen:regenerate-test HIVE-2883
   ```

## Arguments
- **$1** (required): JIRA issue key

## Use Cases
- JIRA issue was updated
- Test case quality needs improvement

## See Also
- `/test-gen:generate-test` - Normal generation
- `/test-gen:regenerate-e2e` - Regenerate E2E
