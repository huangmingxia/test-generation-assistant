---
description: Create a pull request for E2E test code to openshift-tests-private
argument-hint: [JIRA_KEY]
---

## Name
submit-pr

## Synopsis
```
/submit-pr JIRA_KEY
```

## Description
The `submit-pr` command creates a pull request for the E2E test code to the openshift-tests-private repository.

This is the final step in the test generation workflow.

## Implementation
Executes the pr-submitter agent at `config/agents/pr-submitter.md`

The agent performs:
1. Verify E2E test code exists
2. Create commit with proper message format
3. Push to fork repository
4. Create pull request to openshift-tests-private

Total execution time: ~30 seconds

## Return Value
- **Success**: Pull request URL
- **Failure**: Error message indicating submission failure

## Examples

1. **Basic usage**:
   ```
   /submit-pr HIVE-2883
   ```

2. **For different component**:
   ```
   /submit-pr CCO-1234
   ```

## Arguments
- **$1** (required): JIRA issue key (e.g., HIVE-2883, CCO-1234)

## Prerequisites
- E2E test code must exist and be validated
- GitHub CLI (`gh`) installed and authenticated
- Git fork must be configured
- Branch `ai-case-design-{JIRA_KEY}` must exist
- PR submission rules must be followed

## PR Format
**Commit Message**:
```
Add automated test for {JIRA_KEY}

Generated E2E test cases for {JIRA_TITLE}

JIRA: {JIRA_URL}
```

**PR Title**: `Add automated test for {JIRA_KEY}`

**PR Body**: Includes JIRA details, test description, and validation checklist

## Output
- PR URL: https://github.com/openshift/openshift-tests-private/pull/{PR_NUMBER}

## See Also
- `pr-submitter.md` - Agent implementation
- `/run-tests` - Previous step
- PR submission rules: `config/rules/pr_submission_rules.yaml`

---

Execute pr-submitter agent for: **{args}**
