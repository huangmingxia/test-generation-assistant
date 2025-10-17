# HIVE-2923 Test Coverage Matrix

## Scenario 1 - DNSZone Controller Thrashing with Invalid Credentials
| Platform | Cluster Type(s) | Test Type  | Reasoning                   | Priority   | Test Case ID | Status         |
|----------|-----------------|------------|-----------------------------|------------|--------------|----------------|
| AWS | Managed DNS | E2E | Directly validates the root cause fix - measures resourceVersion stability to prove controller no longer thrashes | High | TC-HIVE-2923-001 | ❌ Not Started |


## Scenario 2 - Error Message Scrubbing Verification
| Platform | Cluster Type(s) | Test Type  | Reasoning                   | Priority   | Test Case ID | Status         |
|----------|-----------------|------------|-----------------------------|------------|--------------|----------------|
| AWS | Managed DNS | E2E | Validates RequestID scrubbing in error messages - ensures both "Request ID" and "RequestID" formats are handled | High | TC-HIVE-2923-002 | ❌ Not Started |


## Scenario 3 - AWSPrivateLink Controller Error Handling
| Platform | Cluster Type(s) | Test Type  | Reasoning                   | Priority   | Test Case ID | Status         |
|----------|-----------------|------------|-----------------------------|------------|--------------|----------------|
| AWS | PrivateLink-enabled | E2E | Verifies centralized error scrubber works across multiple controllers (code refactoring validation) | Medium | TC-HIVE-2923-003 | ❌ Not Started |
