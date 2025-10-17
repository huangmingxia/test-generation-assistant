---
description: Regenerate E2E test code (skip prerequisite checks, force regeneration)
argument-hint: [JIRA_KEY]
---

## Name
regenerate-e2e

## Synopsis
```
/regenerate-e2e JIRA_KEY
```

## Description
The `regenerate-e2e` command regenerates E2E test code, **skipping all prerequisite checks** and **forcing regeneration** even if E2E code already exists.

This is the regeneration variant of `/generate-e2e-case`.

## Implementation
Same as `generate-e2e-case` but with regeneration mode enabled:
- **SKIPS** all prerequisite checks
- **FORCES** regeneration even if E2E code exists
- **OVERWRITES** existing E2E test files

Executes the e2e_test_generation_openshift_private agent at `config/agents/e2e_test_generation_openshift_private.md`

## Return Value
- **Success**: Regenerated E2E test code in `temp_repos/openshift-tests-private/`
- **Failure**: Error message indicating which step failed

## Examples

1. **Basic usage**:
   ```
   /regenerate-e2e HIVE-2883
   ```

2. **For different component**:
   ```
   /regenerate-e2e CCO-1234
   ```

## Arguments
- **$1** (required): JIRA issue key (e.g., HIVE-2883, CCO-1234)

## When to Use
- Fix issues in existing E2E code
- Update E2E test based on new requirements
- Regenerate after test case updates
- Force complete regeneration

## Behavior Difference
| Aspect | `/generate-e2e-case` | `/regenerate-e2e` |
|--------|---------------------|------------------|
| Prerequisite check | ✅ Yes | ❌ No (skip) |
| Overwrite existing | ❌ No (skip if exists) | ✅ Yes (force) |
| Use case | First time generation | Update/fix existing |

## Output Location
```
temp_repos/openshift-tests-private/
└── test/extended/{component}/
    └── {jira_key}_test.go (overwritten)
```

Branch: `ai-case-design-{JIRA_KEY}` (recreated if needed)

## See Also
- `/generate-e2e-case` - Normal E2E code generation
- `/regenerate-test` - Regenerate test cases
- `e2e_test_generation_openshift_private.md` - Agent implementation

---

**REGENERATE MODE**: Skip prerequisite checks and force regeneration for: **{args}**
