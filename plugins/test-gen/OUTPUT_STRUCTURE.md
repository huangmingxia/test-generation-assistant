# Test Artifacts Output Structure

## 📁 Directory Structure

All test generation outputs are stored in the `test_artifacts/` directory at the project root.

```
test_artifacts/
└── {JIRA_KEY}/                           # e.g., HIVE-2883, CCO-1234
    ├── phases/                           # Phase-by-phase outputs
    │   ├── test_requirements_output.md   # Phase 1: Requirements analysis
    │   ├── test_strategy.md              # Phase 2: Test strategy
    │   └── test_case_design.md           # Phase 3: Test case design
    │
    ├── test_cases/                       # Final test cases
    │   └── {JIRA_KEY}_test_case.md       # e.g., HIVE-2883_test_case.md
    │
    ├── test_coverage_matrix.md           # Coverage matrix
    ├── test_report.md                    # Test execution report (if tests run)
    └── e2e_test_info.json               # E2E test metadata (if E2E generated)
```

## 📝 Output Path Examples

### Test Case Generation

**Command**: `/generate-test-case HIVE-2883`

**Outputs**:
```
test_artifacts/HIVE-2883/
├── phases/
│   ├── test_requirements_output.md
│   ├── test_strategy.md
│   └── test_case_design.md
├── test_cases/
│   └── HIVE-2883_test_case.md
└── test_coverage_matrix.md
```

### E2E Test Generation

**Command**: `/generate-e2e-case HIVE-2883`

**Prerequisites**: Test case must exist in `test_artifacts/HIVE-2883/test_cases/`

**Outputs**:
- E2E code in `openshift-tests-private` repository
- Metadata in `test_artifacts/HIVE-2883/e2e_test_info.json`

### Test Execution

**Command**: `/run-tests HIVE-2883`

**Outputs**:
```
test_artifacts/HIVE-2883/
└── test_report.md                # Test execution report
```

### Full Workflow

**Command**: `/full-workflow HIVE-2883`

**Outputs** (all combined):
```
test_artifacts/HIVE-2883/
├── phases/
│   ├── test_requirements_output.md
│   ├── test_strategy.md
│   └── test_case_design.md
├── test_cases/
│   └── HIVE-2883_test_case.md
├── test_coverage_matrix.md
├── e2e_test_info.json
└── test_report.md
```

## 🔍 Component-Specific Structure (Legacy)

Some older configurations may use component-based paths:

```
test_artifacts/
└── {component}/                  # e.g., hive, cloudcredential
    └── {JIRA_KEY}/
        └── ...
```

**Note**: The simplified structure (`test_artifacts/{JIRA_KEY}/`) is now preferred.

## ✅ Path Rules

1. **All outputs go to `test_artifacts/`** - Never use `workflow_outputs/` or other directories
2. **JIRA key is the primary folder** - e.g., `test_artifacts/HIVE-2883/`
3. **Phases go in `phases/` subdirectory** - For systematic thinking outputs
4. **Final test cases go in `test_cases/`** - For Polarion-ready test cases
5. **Reports at root level** - `test_report.md`, `test_coverage_matrix.md`

## 🚫 Deprecated Paths

❌ `workflow_outputs/` - Do not use
❌ `test_artifacts/{component}/{JIRA_KEY}/` - Use simplified path instead
❌ Any other custom output directories

## 📦 Example Usage

```bash
# Generate test case
/generate-test-case HIVE-2883
# → test_artifacts/HIVE-2883/test_cases/HIVE-2883_test_case.md

# Generate E2E test (prerequisite: test case exists)
/generate-e2e-case HIVE-2883
# → Checks: test_artifacts/HIVE-2883/test_cases/
# → Creates: E2E code in openshift-tests-private

# Run tests (prerequisite: E2E code exists)
/run-tests HIVE-2883
# → test_artifacts/HIVE-2883/test_report.md

# Full workflow (all in one)
/full-workflow HIVE-2883
# → All outputs in test_artifacts/HIVE-2883/
```

## 🔗 Related Configuration

- Agent configs: `config/agents/*.md` - Define output paths
- Commands: `commands/*.md` - Reference output paths
- Templates: `config/templates/*.yaml` - Template structure for outputs
