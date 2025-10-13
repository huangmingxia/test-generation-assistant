# Generate E2E Test Code

Execute E2E test code generation and integration into openshift-tests-private.

## Usage
```
/generate-e2e HIVE-2883
/generate-e2e CCO-1234
```

## What this command does

1. Check prerequisites (test case must exist)
2. Load e2e_test_generation_openshift_private agent
3. Generate E2E test code
4. Integrate into openshift-tests-private repository
5. Create branch: `ai-case-design-{JIRA_KEY}`

## Arguments
- `$1` (required): JIRA issue key (e.g., HIVE-2883)

## Prerequisite Check
- Checks if test case exists in `workflow_outputs/{COMPONENT}/{JIRA_KEY}/test_cases/`
- If missing, prompts to run `/generate-test` first

## Regenerate Mode
Use `/regenerate-e2e` to skip prerequisite check and force regeneration.

## Example
```
User: /generate-e2e HIVE-2883
→ Checks prerequisite
→ Executes e2e_test_generation_openshift_private agent
→ Output: E2E code in openshift-tests-private
```

---

Execute E2E test generation agent for: **{args}**
