---
description: Submit E2E test code as pull request to openshift-tests-private
argument-hint: [JIRA_KEY]
---

## Name
submit-pr

## Synopsis
```
/test-gen:submit-pr JIRA_KEY
```

## Description
The `submit-pr` command creates and submits a pull request containing the generated E2E test code to the openshift-tests-private repository.

## Implementation
Executes the PR submission agent at `config/agents/pr-submitter.md`

Execution time: ~30 seconds

## Examples
1. **Basic usage**:
   ```
   /test-gen:submit-pr HIVE-2883
   ```

## Arguments
- **$1** (required): JIRA issue key

## Prerequisites
- E2E test code exists
- GitHub CLI (`gh`) installed

## Complete Workflow
1. `/test-gen:generate-test JIRA_KEY`
2. `/test-gen:generate-e2e JIRA_KEY`
3. `/test-gen:run-tests JIRA_KEY`
4. `/test-gen:submit-pr JIRA_KEY` ← This command
