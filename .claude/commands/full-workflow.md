# Full Test Generation Workflow

Execute complete workflow: test case → E2E code → run tests.

## Usage
```
/full-workflow HIVE-2883
/full-workflow CCO-1234
```

## What this command does

Executes all agents in sequence:
1. **test_case_generation** - Generate test cases (90s)
2. **e2e_test_generation_openshift_private** - Generate E2E code
3. **test-executor** - Run E2E tests

**Total time**: ~3-4 minutes for complete workflow

## Arguments
- `$1` (required): JIRA issue key (e.g., HIVE-2883)

## Example
```
User: /full-workflow HIVE-2883
→ Step 1: Generate test case (90s)
→ Step 2: Generate E2E code
→ Step 3: Run tests
→ Output: Complete workflow results
```

---

Execute complete test generation workflow for: **{args}**
