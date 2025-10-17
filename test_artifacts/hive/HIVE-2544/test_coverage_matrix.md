# HIVE-2544 Test Coverage Matrix

## Scenario 1 - Non-Zoned Region MachinePool Creation
| Platform | Cluster Type(s) | Test Type  | Reasoning                   | Priority   | Test Case ID | Status         |
|----------|-----------------|------------|-----------------------------|------------|--------------|----------------|
| Azure    | IPI             | E2E        | Core fix validation - automated creation in non-zoned regions | High       | TC-HIVE-2544-001 | ❌ Not Started |
| Azure Gov | IPI            | Manual     | Government cloud requires manual validation due to access restrictions | High | TC-HIVE-2544-002 | ❌ Not Started |


## Scenario 2 - Zoned Region Regression Test
| Platform | Cluster Type(s) | Test Type  | Reasoning                   | Priority   | Test Case ID | Status         |
|----------|-----------------|------------|-----------------------------|------------|--------------|----------------|
| Azure    | IPI             | E2E        | Regression test - ensure zoned regions still work correctly | High       | TC-HIVE-2544-003 | ❌ Not Started |
