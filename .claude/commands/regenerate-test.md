---
description: Regenerate test case (skip prerequisite checks, force regeneration)
argument-hint: [JIRA_KEY]
---

## Name
regenerate-test

## Synopsis
```
/regenerate-test JIRA_KEY
```

## Description
The `regenerate-test` command regenerates test cases from a JIRA issue, **skipping all prerequisite checks** and **forcing regeneration** even if test cases already exist.

This is the regeneration variant of `/generate-test-case`.

## Implementation
Same as `generate-test-case` but with regeneration mode enabled:
- **SKIPS** all prerequisite checks
- **FORCES** regeneration even if test case exists
- **OVERWRITES** existing test case files

Executes the test_case_generation agent at `config/agents/test_case_generation.md`

## Return Value
- **Success**: Regenerated test artifacts in `test_artifacts/{COMPONENT}/{JIRA_KEY}/`
- **Failure**: Error message indicating which phase failed

## Examples

1. **Basic usage**:
   ```
   /regenerate-test HIVE-2883
   ```

2. **For different component**:
   ```
   /regenerate-test CCO-1234
   ```

## Arguments
- **$1** (required): JIRA issue key (e.g., HIVE-2883, CCO-1234)

## When to Use
- Fix issues in existing test case
- Update test case based on new requirements
- Regenerate after JIRA issue updates
- Force complete regeneration

## Behavior Difference
| Aspect | `/generate-test-case` | `/regenerate-test` |
|--------|----------------------|-------------------|
| Prerequisite check | ✅ Yes | ❌ No (skip) |
| Overwrite existing | ❌ No (skip if exists) | ✅ Yes (force) |
| Use case | First time generation | Update/fix existing |

## Output Structure
```
test_artifacts/{COMPONENT}/{JIRA_KEY}/
├── phases/
│   ├── test_requirements_output.md (overwritten)
│   └── test_strategy.md (overwritten)
├── test_cases/
│   └── {JIRA_KEY}_test_cases.md (overwritten)
└── test_coverage_matrix.md (overwritten)
```

## See Also
- `/generate-test-case` - Normal test case generation
- `/regenerate-e2e` - Regenerate E2E code
- `test_case_generation.md` - Agent implementation

---

**REGENERATE MODE**: Skip prerequisite checks and force regeneration for: **{args}**
