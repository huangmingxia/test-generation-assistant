# Test Strategy

## Test Coverage Matrix

| Scenario | Platform | Cluster Type | Test Type | Reasoning | Priority | Test Case ID | Status |
|----------|----------|--------------|-----------|-----------|----------|--------------|--------|
| DNSZone Controller Thrashing with Invalid Credentials | AWS | Managed DNS | E2E | Directly validates the root cause fix - measures resourceVersion stability to prove controller no longer thrashes | High | TC-HIVE-2923-001 | ❌ Not Started |
| Error Message Scrubbing Verification | AWS | Managed DNS | E2E | Validates RequestID scrubbing in error messages - ensures both "Request ID" and "RequestID" formats are handled | High | TC-HIVE-2923-002 | ❌ Not Started |
| AWSPrivateLink Controller Error Handling | AWS | PrivateLink-enabled | E2E | Verifies centralized error scrubber works across multiple controllers (code refactoring validation) | Medium | TC-HIVE-2923-003 | ❌ Not Started |

## Test Scenarios

### Scenario 1: DNSZone Controller Thrashing with Invalid Credentials
**Validation Type**: Type A (Stability)

**Objective**: Verify that DNSZone controller no longer thrashes when encountering AWS errors with RequestID

**User Workflow**:
1. User attempts to create a managed DNS cluster with invalid AWS credentials
2. AWS API returns authentication errors containing RequestID
3. DNSZone controller processes errors and updates status conditions
4. System should maintain stable state without continuous reconciliation

**Quantitative Validation**:
- **Metric**: DNSZone resourceVersion change rate
- **Pre-fix Behavior**: resourceVersion increments ~348 times in 30 seconds (11.6 changes/second)
- **Post-fix Behavior**: resourceVersion increments 0 times in 30 seconds (stable)
- **Threshold**: resourceVersion changes ≤ 1 in 30-second window = PASS
- **Measurement Method**:
  ```bash
  INITIAL_RV=$(oc get dnszone <name> -o jsonpath='{.metadata.resourceVersion}')
  sleep 30
  FINAL_RV=$(oc get dnszone <name> -o jsonpath='{.metadata.resourceVersion}')
  CHANGE=$((FINAL_RV - INITIAL_RV))
  ```

**Expected Results**:
- ResourceVersion remains stable (change count ≤ 1)
- Error messages contain scrubbed RequestID ("XXXX")
- Controller logs show single reconciliation attempt, not continuous loop
- No exponential backoff warnings in controller logs

---

### Scenario 2: Error Message Scrubbing Verification
**Validation Type**: Type D (Absence)

**Objective**: Confirm that RequestID values (UUID format) are absent from DNSZone status conditions

**User Workflow**:
1. User creates ClusterDeployment with managed DNS using invalid credentials
2. AWS returns errors with RequestID in various formats
3. Controller updates DNSZone status condition with scrubbed error message
4. User inspects DNSZone status to understand failure

**Quantitative Validation**:
- **Metric**: Presence of UUID patterns in DNSZone status.conditions[type=DNSError].message
- **Pattern to Verify Absent**: `[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}` (UUID format)
- **Pattern to Verify Present**: `request ?id: XXXX` or `RequestID: XXXX` (scrubbed format, case-insensitive)
- **Threshold**: 0 UUID matches in error message = PASS
- **Measurement Method**:
  ```bash
  ERROR_MSG=$(oc get dnszone <name> -o jsonpath='{.status.conditions[?(@.type=="DNSError")].message}')
  # Verify no UUIDs present
  echo "$ERROR_MSG" | grep -E '[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}' && echo "FAIL" || echo "PASS"
  # Verify scrubbed format present
  echo "$ERROR_MSG" | grep -iE 'request ?id: XXXX' && echo "PASS" || echo "FAIL"
  ```

**Test Variations**:
1. Single RequestID with space: `Request ID: <uuid>`
2. Single RequestID without space: `RequestID: <uuid>`
3. Multiple RequestIDs in one message (different formats)
4. RequestID at end of message
5. Embedded newlines with RequestID

**Expected Results**:
- 0 UUID patterns found in error messages
- Error messages contain placeholder "XXXX" instead of actual RequestID
- Error context and details remain intact and readable
- Both "Request ID" and "RequestID" formats properly scrubbed

---

### Scenario 3: AWSPrivateLink Controller Error Handling
**Validation Type**: Type D (Absence)

**Objective**: Verify centralized error scrubber works consistently across AWSPrivateLink and PrivateLink controllers

**User Workflow**:
1. User creates ClusterDeployment with AWS PrivateLink enabled
2. Configure unauthorized AWS IAM permissions
3. Controller attempts VPC endpoint operations
4. AWS returns UnauthorizedOperation errors with RequestID
5. User checks ClusterDeployment conditions for error details

**Quantitative Validation**:
- **Metric**: Presence of UUID patterns in ClusterDeployment status conditions (AWSPrivateLinkFailed, PrivateLinkFailed)
- **Pattern to Verify Absent**: UUID format `[0-9a-f-]{36}`
- **Threshold**: 0 UUID matches across all relevant status conditions = PASS
- **Measurement Method**:
  ```bash
  # Check AWSPrivateLinkFailed condition
  oc get clusterdeployment <name> -o jsonpath='{.status.conditions[?(@.type=="AWSPrivateLinkFailed")].message}' | grep -E '[0-9a-f-]{36}' && echo "FAIL" || echo "PASS"
  # Check PrivateLinkFailed condition
  oc get clusterdeployment <name> -o jsonpath='{.status.conditions[?(@.type=="PrivateLinkFailed")].message}' | grep -E '[0-9a-f-]{36}' && echo "FAIL" || echo "PASS"
  ```

**Code Refactoring Validation**:
- Confirm `controllerutils.ErrorScrub()` is used (not local regex functions)
- Verify duplicate `filterErrorMessage()` functions removed from:
  - `pkg/controller/awsprivatelink/awsprivatelink_controller.go`
  - `pkg/controller/privatelink/conditions/conditions.go`

**Expected Results**:
- No UUID values in AWSPrivateLinkFailed or PrivateLinkFailed condition messages
- Error messages use centralized scrubbing utility
- Consistent scrubbing behavior across all controllers
- Test coverage exists for centralized utility function

## Validation Methods

### 1. ResourceVersion Stability Measurement
**Method**: Time-series comparison of Kubernetes resource metadata
**Implementation**:
```bash
# Capture initial state
INITIAL_RV=$(oc get dnszone <name> -o jsonpath='{.metadata.resourceVersion}')
INITIAL_GENERATION=$(oc get dnszone <name> -o jsonpath='{.metadata.generation}')
INITIAL_TIME=$(date +%s)

# Wait observation period
sleep 30

# Capture final state
FINAL_RV=$(oc get dnszone <name> -o jsonpath='{.metadata.resourceVersion}')
FINAL_GENERATION=$(oc get dnszone <name> -o jsonpath='{.metadata.generation}')
FINAL_TIME=$(date +%s)

# Calculate metrics
RV_CHANGE=$((FINAL_RV - INITIAL_RV))
DURATION=$((FINAL_TIME - INITIAL_TIME))

# Evaluate threshold
if [ $RV_CHANGE -le 1 ]; then
  echo "PASS: Stable (resourceVersion changed $RV_CHANGE times in ${DURATION}s)"
else
  echo "FAIL: Thrashing (resourceVersion changed $RV_CHANGE times in ${DURATION}s)"
fi
```

**Threshold**: ≤ 1 resourceVersion change in 30 seconds = stable behavior

---

### 2. Error Message Content Analysis
**Method**: Pattern matching and regex validation of status condition messages
**Implementation**:
```bash
# Extract error message from DNSZone status
ERROR_MSG=$(oc get dnszone <name> -o jsonpath='{.status.conditions[?(@.type=="DNSError")].message}')

# Test 1: Verify UUIDs are absent
UUID_COUNT=$(echo "$ERROR_MSG" | grep -oE '[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}' | wc -l)
if [ $UUID_COUNT -eq 0 ]; then
  echo "PASS: No UUIDs found in error message"
else
  echo "FAIL: Found $UUID_COUNT UUID(s) in error message"
fi

# Test 2: Verify scrubbed placeholder is present
if echo "$ERROR_MSG" | grep -qiE 'request ?id: XXXX'; then
  echo "PASS: Scrubbed RequestID placeholder found"
else
  echo "FAIL: No scrubbed RequestID placeholder found"
fi

# Test 3: Verify error context is preserved
if echo "$ERROR_MSG" | grep -qE '(AccessDenied|UnauthorizedOperation|status code: 403)'; then
  echo "PASS: Error context preserved"
else
  echo "FAIL: Error context missing"
fi
```

**Threshold**: 0 UUIDs + scrubbed placeholder present + error context intact = PASS

---

### 3. Controller Reconciliation Frequency
**Method**: Log analysis of controller reconciliation events
**Implementation**:
```bash
# Get DNSZone controller pod
CONTROLLER_POD=$(oc get pods -n hive -l control-plane=controller-manager -o jsonpath='{.items[0].metadata.name}')

# Monitor reconciliation logs for specific DNSZone
DNSZONE_NAME="<cluster-name>-zone"
START_TIME=$(date +%s)

# Collect logs for 60 seconds
oc logs -n hive $CONTROLLER_POD --since=60s | grep -c "Reconciling DNSZone.*$DNSZONE_NAME" > /tmp/reconcile_count.txt

# Calculate reconciliation rate
END_TIME=$(date +%s)
DURATION=$((END_TIME - START_TIME))
RECON_COUNT=$(cat /tmp/reconcile_count.txt)
RATE=$(echo "scale=2; $RECON_COUNT / $DURATION" | bc)

# Evaluate threshold
if (( $(echo "$RATE < 0.1" | bc -l) )); then
  echo "PASS: Normal reconciliation rate ($RATE reconciles/second)"
else
  echo "FAIL: Excessive reconciliation rate ($RATE reconciles/second)"
fi
```

**Threshold**: < 0.1 reconciles/second = normal behavior (not thrashing)

---

### 4. Multi-Controller Consistency Validation
**Method**: Cross-controller error scrubbing verification
**Implementation**:
```bash
# Collect error messages from multiple controllers
DNSZONE_ERROR=$(oc get dnszone <name> -o jsonpath='{.status.conditions[?(@.type=="DNSError")].message}')
PL_ERROR=$(oc get clusterdeployment <name> -o jsonpath='{.status.conditions[?(@.type=="PrivateLinkFailed")].message}')
AWSPL_ERROR=$(oc get clusterdeployment <name> -o jsonpath='{.status.conditions[?(@.type=="AWSPrivateLinkFailed")].message}')

# Function to check UUID presence
check_scrubbing() {
  local msg="$1"
  local controller="$2"
  local uuid_count=$(echo "$msg" | grep -oE '[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}' | wc -l)

  if [ $uuid_count -eq 0 ]; then
    echo "PASS: $controller - No UUIDs found"
    return 0
  else
    echo "FAIL: $controller - Found $uuid_count UUID(s)"
    return 1
  fi
}

# Validate all controllers
PASS_COUNT=0
check_scrubbing "$DNSZONE_ERROR" "DNSZone" && ((PASS_COUNT++))
check_scrubbing "$PL_ERROR" "PrivateLink" && ((PASS_COUNT++))
check_scrubbing "$AWSPL_ERROR" "AWSPrivateLink" && ((PASS_COUNT++))

# Overall result
if [ $PASS_COUNT -eq 3 ]; then
  echo "PASS: All controllers use consistent error scrubbing"
else
  echo "FAIL: Inconsistent error scrubbing across controllers"
fi
```

**Threshold**: 100% of controllers (3/3) must pass scrubbing validation = PASS
