# Regenerate Test Case

Regenerate test case (skip prerequisite checks, force regeneration).

## Usage
```
/regenerate-test HIVE-2883
/regenerate-test CCO-1234
```

## What this command does

Same as `/generate-test` but:
- **SKIPS** all prerequisite checks
- **FORCES** regeneration even if test case exists
- **OVERWRITES** existing test case files

## When to use
- Fix issues in existing test case
- Update test case based on new requirements
- Regenerate after JIRA issue updates

## Example
```
User: /regenerate-test HIVE-2883
→ Executes test_case_generation agent (SKIP prerequisite check)
→ Overwrites existing test case
```

---

**REGENERATE MODE**: Skip prerequisite checks and force regeneration for: **{args}**
