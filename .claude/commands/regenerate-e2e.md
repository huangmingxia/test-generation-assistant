# Regenerate E2E Test Code

Regenerate E2E test code (skip prerequisite checks, force regeneration).

## Usage
```
/regenerate-e2e HIVE-2883
/regenerate-e2e CCO-1234
```

## What this command does

Same as `/generate-e2e` but:
- **SKIPS** prerequisite check (test case existence)
- **FORCES** E2E code regeneration
- **OVERWRITES** existing E2E test files

## When to use
- Fix issues in existing E2E code
- Update E2E code based on new implementation
- Regenerate after code review feedback

## Example
```
User: /regenerate-e2e HIVE-2883
→ Executes e2e_test_generation_openshift_private agent (SKIP prerequisite)
→ Overwrites existing E2E code
```

---

**REGENERATE MODE**: Skip E2E prerequisite check and force regeneration for: **{args}**
