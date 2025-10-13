# Generate Test Case

Execute the complete test case generation workflow for a JIRA issue.

## Usage
```
/generate-test HIVE-2883
/generate-test CCO-1234
```

## What this command does

1. **MANDATORY**: Execute test_case_generation agent `config/agents/test_case_generation.md` configuration

## Arguments
- `$1` (required): JIRA issue key (e.g., HIVE-2883, CCO-1234)

## Execution Protocol

**CRITICAL**: Execute with maximum efficiency:
- ✅ Read `config/agents/test_case_generation.yaml`
- ✅ Execute EACH step in agent config manually
- ✅ Verify each step output before proceeding
- ✅ Use EXACT commands from config
- ❌ NEVER skip steps

## Example
```
User: /generate-test HIVE-2883
→ Executes test_case_generation agent for HIVE-2883
→ Output: test_artifacts/hive/HIVE-2883/
```

---

Execute test case generation agent for: **{args}**
