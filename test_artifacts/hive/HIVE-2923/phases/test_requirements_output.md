# Test Requirements Analysis

## Component Name
hive

## Card Summary
**HIVE-2923: Scrub AWS RequestID from DNSZone "DNSError" status condition message (thrashing)**

The issue addresses a controller thrashing problem in OpenShift Hive where the DNSZone and ClusterDeployment controllers were experiencing high error rates without backoff. The root cause was that the AWS RequestID (a per-request UUID) in the DNSError status condition message was changing with every API call, causing unnecessary CR updates and immediate requeues.

The fix involves updating the regex pattern in the error scrubber utility to handle the new AWS spelling format `RequestID` (without a space between "Request" and "ID"), which was previously `Request ID` (with a space). The change was likely introduced by the AWS SDK v2 transition or an AWS API change.

## Test Requirements

### Functional Requirements
1. **Error Message Scrubbing**: Verify that AWS RequestID values are properly scrubbed from error messages in DNSZone status conditions
2. **Regex Pattern Matching**: Confirm that the updated regex pattern `request ?id:` correctly matches both `request id:` and `RequestID:` formats (case-insensitive)
3. **Controller Stability**: Ensure that DNSZone and ClusterDeployment controllers no longer thrash when encountering AWS errors with RequestID
4. **Resource Version Stability**: Verify that DNSZone CR resourceVersion remains stable when encountering repeated AWS errors (not incrementing unnecessarily)
5. **Error Message Consistency**: Confirm that scrubbed error messages maintain readability while removing sensitive/transient data

### Technical Requirements
1. **Error Scrubber Utility**: Test the `controllerutils.ErrorScrub()` function with various AWS error message formats
2. **DRY Code Refactoring**: Verify that duplicate regex-based scrubbing code has been consolidated into the centralized utility
3. **Multiple Controllers**: Test error scrubbing across DNSZone, AWSPrivateLink, and PrivateLink controllers
4. **Test Coverage**: Ensure unit tests cover all RequestID format variations (with/without space, different cases)

### Integration Requirements
1. **Invalid Credentials Scenario**: Test with invalid AWS credentials to trigger RequestID errors
2. **AWS SDK v2 Compatibility**: Verify compatibility with AWS SDK v2 error message formats
3. **Controller Reconciliation**: Confirm that reconciliation loops don't trigger unnecessarily after the fix

## Affected Platforms
- **Cloud Platform**: AWS only
- **OpenShift Version**: OpenShift 4.20+
- **Component**: Hive DNSZone controller, AWSPrivateLink controller, PrivateLink conditions

## Test Scenarios

### Scenario 1: DNSZone Controller with Invalid Credentials
**Objective**: Verify that DNSZone controller no longer thrashes when AWS returns errors with RequestID

**Setup**:
- Create a ClusterDeployment with managed DNS enabled
- Configure invalid AWS credentials
- Monitor DNSZone CR resourceVersion over time

**Expected Behavior**:
- Before fix: resourceVersion increments rapidly (e.g., 348 changes in 30 seconds)
- After fix: resourceVersion remains stable (0 changes in 30 seconds)
- Error messages should have RequestID scrubbed to "XXXX"

### Scenario 2: Error Message Format Variations
**Objective**: Confirm regex pattern handles all RequestID format variations

**Test Cases**:
- `Request ID: <uuid>` (space, title case)
- `RequestID: <uuid>` (no space, camel case)
- `request id: <uuid>` (space, lowercase)
- Multiple RequestIDs in single error message

**Expected Behavior**:
- All RequestID values replaced with placeholder
- Error message structure preserved
- Other error details remain intact

### Scenario 3: AWSPrivateLink Controller Error Scrubbing
**Objective**: Verify centralized error scrubbing works across multiple controllers

**Setup**:
- Test AWSPrivateLink controller with unauthorized AWS operations
- Trigger errors containing RequestID

**Expected Behavior**:
- Errors use centralized `controllerutils.ErrorScrub()` function
- RequestID values properly scrubbed
- No duplicate regex implementations

### Scenario 4: Unit Test Coverage
**Objective**: Ensure comprehensive test coverage for error scrubber

**Test Cases**:
- Single RequestID at end of message
- Two RequestIDs with different spellings
- Embedded newlines in error messages
- Edge cases from AWS SDK v2 error formats

**Expected Behavior**:
- All test cases pass with correct scrubbing behavior
- Test cases added in PR #2744 provide adequate coverage

## Edge Cases

1. **Multiple RequestID Formats in Single Message**: Error contains both `Request ID:` and `RequestID:` formats
2. **RequestID at Message Boundary**: RequestID appears at the beginning or end of error message
3. **Nested Error Messages**: Multi-line errors with multiple RequestIDs
4. **Concurrent Controller Reconciliations**: Multiple controllers processing errors simultaneously
5. **AWS SDK Version Transitions**: Error format changes during SDK upgrades
6. **Non-AWS Controllers**: Verify fix doesn't affect non-AWS error handling
7. **Empty or Malformed RequestIDs**: Handle edge cases where RequestID format is unexpected
8. **Performance Impact**: Ensure regex pattern performance is acceptable for high-frequency reconciliation
