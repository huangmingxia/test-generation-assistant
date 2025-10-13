# Submit Pull Request

Create a pull request for E2E test code to openshift-tests-private.

## Usage
```
/submit-e2e-pr HIVE-2883
/submit-e2e-pr CCO-1234
```

## What this command does

1. Load pr-submitter agent configuration
2. Verify E2E test code exists
3. Create commit with proper message format
4. Push to fork repository
5. Create pull request to openshift-tests-private

## Arguments
- `$1` (required): JIRA issue key (e.g., HIVE-2883)

## Prerequisites
- E2E test code must exist and be validated
- Git fork must be configured
- Branch `ai-case-design-{JIRA_KEY}` must exist

## Example
```
User: /submit-pr HIVE-2883
→ Executes pr-submitter agent
→ Creates commit and PR
→ Output: PR URL
```

---

Execute pr-submitter agent for: **{args}**
