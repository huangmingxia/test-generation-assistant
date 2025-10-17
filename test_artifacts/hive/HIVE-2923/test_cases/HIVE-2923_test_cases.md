# Test Case: HIVE-2923
**Component:** hive
**Summary:** Scrub AWS RequestID from DNSZone "DNSError" status condition message (thrashing)

## Test Overview
- **Total Test Cases:** 3
- **Test Types:** E2E (3)
- **Estimated Time:** 45-60 minutes

## Test Cases

### Test Case HIVE-2923_001
**Name:** DNSZone Controller Stability with Invalid AWS Credentials
**Description:** Verify that the DNSZone controller maintains stable behavior (no thrashing) when encountering AWS authentication errors containing RequestID in the new format.
**Type:** E2E
**Priority:** High

#### Prerequisites
- OpenShift Hive operator deployed and running (4.20+)
- AWS account with ability to create invalid credentials
- Access to monitor Hive controller logs
- oc CLI configured with cluster-admin access

#### Test Steps
1. **Action:** Create a ClusterDeployment with managed DNS using intentionally invalid AWS credentials
   ```bash
   # Create secret with invalid AWS credentials
   oc create secret generic invalid-aws-creds \
     --from-literal=aws_access_key_id=INVALID_KEY_ID \
     --from-literal=aws_secret_access_key=INVALID_SECRET_KEY \
     -n <namespace>

   # Create ClusterDeployment with managed DNS enabled
   cat <<EOF | oc apply -f -
   apiVersion: hive.openshift.io/v1
   kind: ClusterDeployment
   metadata:
     name: test-dnszone-stability
     namespace: <namespace>
   spec:
     baseDomain: example.com
     clusterName: test-cluster
     platform:
       aws:
         region: us-east-1
         credentialsSecretRef:
           name: invalid-aws-creds
     manageDNS: true
   EOF
   ```
   **Expected:** ClusterDeployment and associated DNSZone created successfully (status may show errors due to invalid credentials)

2. **Action:** Wait for DNSZone controller to attempt DNS operations and capture initial resourceVersion
   ```bash
   # Wait for DNSZone to be created
   sleep 10

   # Capture initial resourceVersion
   DNSZONE_NAME=$(oc get dnszone -n <namespace> -o name | grep test-cluster)
   INITIAL_RV=$(oc get $DNSZONE_NAME -n <namespace> -o jsonpath='{.metadata.resourceVersion}')
   echo "Initial resourceVersion: $INITIAL_RV"
   ```
   **Expected:** DNSZone exists with initial resourceVersion captured

3. **Action:** Monitor DNSZone resourceVersion over a 30-second observation period to measure stability
   ```bash
   # Wait observation period
   sleep 30

   # Capture final resourceVersion
   FINAL_RV=$(oc get $DNSZONE_NAME -n <namespace> -o jsonpath='{.metadata.resourceVersion}')
   RV_CHANGE=$((FINAL_RV - INITIAL_RV))

   echo "Final resourceVersion: $FINAL_RV"
   echo "ResourceVersion change: $RV_CHANGE"
   ```
   **Expected:** resourceVersion change ≤ 1 (indicating stable behavior, not thrashing)

4. **Action:** Verify DNSZone status condition contains scrubbed error message
   ```bash
   # Extract DNSError condition message
   ERROR_MSG=$(oc get $DNSZONE_NAME -n <namespace> \
     -o jsonpath='{.status.conditions[?(@.type=="DNSError")].message}')

   echo "Error message: $ERROR_MSG"

   # Check for UUID presence (should be absent)
   UUID_COUNT=$(echo "$ERROR_MSG" | grep -oE '[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}' | wc -l)
   echo "UUID count: $UUID_COUNT (expected: 0)"

   # Check for scrubbed placeholder (should be present)
   if echo "$ERROR_MSG" | grep -qiE 'request ?id:'; then
     echo "Scrubbed RequestID format found: PASS"
   else
     echo "Scrubbed RequestID format NOT found: FAIL"
   fi
   ```
   **Expected:**
   - UUID count = 0 (no actual RequestID UUIDs present)
   - Scrubbed format present (e.g., "request id: XXXX" or "RequestID: XXXX")
   - Error message readable and contains context about authentication failure

5. **Action:** Examine Hive controller logs to verify reconciliation frequency
   ```bash
   # Get Hive controller pod
   CONTROLLER_POD=$(oc get pods -n hive -l control-plane=controller-manager \
     -o jsonpath='{.items[0].metadata.name}')

   # Count reconciliation attempts in last 60 seconds
   RECON_COUNT=$(oc logs -n hive $CONTROLLER_POD --since=60s | \
     grep -c "Reconciling DNSZone.*test-cluster" || echo 0)

   echo "Reconciliation count in 60s: $RECON_COUNT"
   RATE=$(echo "scale=2; $RECON_COUNT / 60" | bc)
   echo "Reconciliation rate: $RATE per second"
   ```
   **Expected:**
   - Reconciliation rate < 0.1 per second (indicates normal backoff, not thrashing)
   - No continuous reconciliation loop visible in logs

6. **Action:** Cleanup test resources
   ```bash
   oc delete clusterdeployment test-dnszone-stability -n <namespace>
   oc delete secret invalid-aws-creds -n <namespace>
   ```
   **Expected:** Resources deleted successfully

---

### Test Case HIVE-2923_002
**Name:** Error Message RequestID Scrubbing Format Variations
**Description:** Confirm that the updated regex pattern correctly scrubs RequestID in both old format ("Request ID: <uuid>") and new format ("RequestID: <uuid>") from AWS error messages
**Type:** E2E
**Priority:** High

#### Prerequisites
- OpenShift Hive operator deployed (4.20+)
- Access to Hive error scrubber utility source code or test suite
- Go 1.23+ for running unit tests
- Clone of openshift/hive repository (for verification)

#### Test Steps
1. **Action:** Run unit tests for the error scrubber utility to verify regex pattern handling
   ```bash
   # Navigate to hive repository
   cd /path/to/openshift/hive

   # Run error scrubber unit tests
   go test -v ./pkg/controller/utils/errorscrub_test.go \
     -run TestErrorScrubber
   ```
   **Expected:** All test cases pass, including:
   - "suddenly it is spelled RequestID" test case (new format without space)
   - "two request IDs, spelled differently" test case (mixed formats)
   - "embedded newline" test case
   - "request id is at the end" test case

2. **Action:** Verify the regex pattern in the error scrubber source code
   ```bash
   # Check the current regex pattern
   grep -A 1 "awsRequestIDRE" pkg/controller/utils/errorscrub.go
   ```
   **Expected:** Pattern should be: `awsRequestIDRE = regexp.MustCompile(\`(, )*(?i)(request ?id: )(?:[-[:xdigit:]]+)\`)`
   - Notice the optional space `?` in `request ?id:`
   - Case-insensitive flag `(?i)` present

3. **Action:** Create a live test scenario with invalid credentials triggering actual AWS error responses
   ```bash
   # Use same setup as TC-001 but focus on error message content analysis
   # Create DNSZone with invalid credentials
   oc apply -f - <<EOF
   apiVersion: hive.openshift.io/v1
   kind: DNSZone
   metadata:
     name: test-scrubbing-formats
     namespace: <namespace>
   spec:
     zone: test.example.com
     aws:
       credentialsSecretRef:
         name: invalid-aws-creds
       region: us-east-1
   EOF

   # Wait for AWS API calls to occur
   sleep 15
   ```
   **Expected:** DNSZone created with error conditions populated

4. **Action:** Extract and analyze error messages for multiple RequestID occurrences
   ```bash
   # Get full error message
   ERROR_MSG=$(oc get dnszone test-scrubbing-formats -n <namespace> \
     -o jsonpath='{.status.conditions[?(@.type=="DNSError")].message}')

   echo "=== Full Error Message ==="
   echo "$ERROR_MSG"
   echo "=========================="

   # Test for UUID format (36-character format)
   echo -e "\n=== UUID Pattern Check ==="
   if echo "$ERROR_MSG" | grep -qE '[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}'; then
     echo "FAIL: Found UUID in error message"
     echo "$ERROR_MSG" | grep -oE '[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}'
   else
     echo "PASS: No UUIDs found"
   fi

   # Test for scrubbed format (case-insensitive)
   echo -e "\n=== Scrubbed Format Check ==="
   if echo "$ERROR_MSG" | grep -iE 'request ?id:'; then
     echo "PASS: Scrubbed RequestID format found"
     echo "$ERROR_MSG" | grep -ioE 'request ?id: [^,\)]*'
   else
     echo "FAIL: No scrubbed RequestID format found"
   fi

   # Verify error context preserved
   echo -e "\n=== Error Context Check ==="
   if echo "$ERROR_MSG" | grep -qE '(status code:|error|failed|denied)'; then
     echo "PASS: Error context preserved"
   else
     echo "FAIL: Error context missing"
   fi
   ```
   **Expected:**
   - No UUID patterns detected
   - Scrubbed format detected (may be "Request ID:", "RequestID:", or "request id:")
   - Error context (status codes, error descriptions) intact

5. **Action:** Test edge case with multiple RequestID occurrences in single error
   ```bash
   # This requires examining actual AWS SDK v2 multi-line error responses
   # Check if error message has been properly scrubbed even with nested errors

   # Look for patterns like:
   # "... RequestID: XXXX ... status code: 403, request id: XXXX"
   SCRUBBED_COUNT=$(echo "$ERROR_MSG" | grep -ioE 'request ?id: XXXX' | wc -l)
   echo "Scrubbed RequestID placeholders found: $SCRUBBED_COUNT"
   ```
   **Expected:** If multiple RequestIDs were present in original AWS error, all should be scrubbed to "XXXX"

6. **Action:** Cleanup test resources
   ```bash
   oc delete dnszone test-scrubbing-formats -n <namespace>
   ```
   **Expected:** Resources deleted successfully

---

### Test Case HIVE-2923_003
**Name:** Centralized Error Scrubber Across Multiple Controllers
**Description:** Verify that the DRY code refactoring successfully consolidated error scrubbing logic, and that AWSPrivateLink and PrivateLink controllers use the centralized `controllerutils.ErrorScrub()` function consistently
**Type:** E2E
**Priority:** Medium

#### Prerequisites
- OpenShift Hive operator deployed (4.20+)
- AWS account with PrivateLink capabilities
- Access to Hive source code repository
- Ability to configure IAM permissions (for triggering unauthorized errors)
- oc CLI with cluster-admin access

#### Test Steps
1. **Action:** Verify code refactoring by checking that duplicate regex functions were removed
   ```bash
   # Navigate to hive repository
   cd /path/to/openshift/hive

   # Check that filterErrorMessage functions were removed from controllers
   echo "=== Checking AWSPrivateLink Controller ==="
   if grep -q "func filterErrorMessage" pkg/controller/awsprivatelink/awsprivatelink_controller.go; then
     echo "FAIL: filterErrorMessage still exists in awsprivatelink_controller.go"
   else
     echo "PASS: filterErrorMessage removed from awsprivatelink_controller.go"
   fi

   echo -e "\n=== Checking PrivateLink Conditions ==="
   if grep -q "func filterErrorMessage" pkg/controller/privatelink/conditions/conditions.go; then
     echo "FAIL: filterErrorMessage still exists in conditions.go"
   else
     echo "PASS: filterErrorMessage removed from conditions.go"
   fi

   # Verify usage of centralized function
   echo -e "\n=== Verifying Centralized Function Usage ==="
   grep -n "controllerutils.ErrorScrub" pkg/controller/awsprivatelink/awsprivatelink_controller.go
   grep -n "controllerutils.ErrorScrub" pkg/controller/privatelink/conditions/conditions.go
   ```
   **Expected:**
   - Local `filterErrorMessage` functions removed from both files
   - `controllerutils.ErrorScrub()` used instead

2. **Action:** Create ClusterDeployment with AWS PrivateLink enabled using unauthorized credentials
   ```bash
   # Create secret with limited AWS IAM permissions (no VPC endpoint permissions)
   oc create secret generic limited-aws-creds \
     --from-literal=aws_access_key_id=$LIMITED_ACCESS_KEY \
     --from-literal=aws_secret_access_key=$LIMITED_SECRET_KEY \
     -n <namespace>

   # Create ClusterDeployment with PrivateLink enabled
   cat <<EOF | oc apply -f -
   apiVersion: hive.openshift.io/v1
   kind: ClusterDeployment
   metadata:
     name: test-privatelink-scrubbing
     namespace: <namespace>
   spec:
     baseDomain: example.com
     clusterName: test-pl-cluster
     platform:
       aws:
         region: us-east-1
         credentialsSecretRef:
           name: limited-aws-creds
         privateLink:
           enabled: true
   EOF
   ```
   **Expected:** ClusterDeployment created successfully

3. **Action:** Wait for PrivateLink controllers to attempt AWS operations and encounter authorization errors
   ```bash
   # Monitor ClusterDeployment conditions
   sleep 20

   # Check for PrivateLink-related error conditions
   oc get clusterdeployment test-privatelink-scrubbing -n <namespace> \
     -o jsonpath='{.status.conditions[?(@.type=="AWSPrivateLinkFailed")]}{"\n"}{.status.conditions[?(@.type=="PrivateLinkFailed")]}' | jq .
   ```
   **Expected:** Error conditions populated with AWS authorization errors

4. **Action:** Verify AWSPrivateLinkFailed condition has properly scrubbed error message
   ```bash
   # Extract AWSPrivateLinkFailed condition message
   AWSPL_ERROR=$(oc get clusterdeployment test-privatelink-scrubbing -n <namespace> \
     -o jsonpath='{.status.conditions[?(@.type=="AWSPrivateLinkFailed")].message}')

   echo "=== AWSPrivateLinkFailed Error Message ==="
   echo "$AWSPL_ERROR"

   # Check for UUID presence
   UUID_COUNT=$(echo "$AWSPL_ERROR" | grep -oE '[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}' | wc -l)

   if [ $UUID_COUNT -eq 0 ]; then
     echo "PASS: AWSPrivateLink - No UUIDs found"
   else
     echo "FAIL: AWSPrivateLink - Found $UUID_COUNT UUID(s)"
   fi

   # Verify scrubbed format present
   if echo "$AWSPL_ERROR" | grep -qiE 'request ?id:'; then
     echo "PASS: AWSPrivateLink - Scrubbed format present"
   else
     echo "INFO: AWSPrivateLink - No RequestID in this error (may be different AWS error type)"
   fi
   ```
   **Expected:**
   - 0 UUIDs found in error message
   - If RequestID was in original error, scrubbed format present

5. **Action:** Verify PrivateLinkFailed condition has properly scrubbed error message
   ```bash
   # Extract PrivateLinkFailed condition message
   PL_ERROR=$(oc get clusterdeployment test-privatelink-scrubbing -n <namespace> \
     -o jsonpath='{.status.conditions[?(@.type=="PrivateLinkFailed")].message}')

   echo "=== PrivateLinkFailed Error Message ==="
   echo "$PL_ERROR"

   # Check for UUID presence
   UUID_COUNT=$(echo "$PL_ERROR" | grep -oE '[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}' | wc -l)

   if [ $UUID_COUNT -eq 0 ]; then
     echo "PASS: PrivateLink - No UUIDs found"
   else
     echo "FAIL: PrivateLink - Found $UUID_COUNT UUID(s)"
   fi

   # Verify scrubbed format present
   if echo "$PL_ERROR" | grep -qiE 'request ?id:'; then
     echo "PASS: PrivateLink - Scrubbed format present"
   else
     echo "INFO: PrivateLink - No RequestID in this error (may be different AWS error type)"
   fi
   ```
   **Expected:**
   - 0 UUIDs found in error message
   - If RequestID was in original error, scrubbed format present

6. **Action:** Perform consistency check across all controllers using centralized scrubber
   ```bash
   # Collect errors from DNSZone, AWSPrivateLink, and PrivateLink
   echo "=== Multi-Controller Consistency Check ==="

   # Create test function
   check_scrubbing() {
     local msg="$1"
     local controller="$2"
     local uuid_count=$(echo "$msg" | grep -oE '[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}' | wc -l)

     if [ -z "$msg" ]; then
       echo "$controller: No error message (SKIP)"
       return 2
     elif [ $uuid_count -eq 0 ]; then
       echo "$controller: PASS (No UUIDs found)"
       return 0
     else
       echo "$controller: FAIL (Found $uuid_count UUID(s))"
       return 1
     fi
   }

   # Check all controllers
   PASS_COUNT=0
   TOTAL_COUNT=0

   # DNSZone check (if exists from previous test)
   DNSZONE_ERROR=$(oc get dnszone -n <namespace> -o jsonpath='{.items[0].status.conditions[?(@.type=="DNSError")].message}' 2>/dev/null || echo "")
   if [ -n "$DNSZONE_ERROR" ]; then
     check_scrubbing "$DNSZONE_ERROR" "DNSZone" && ((PASS_COUNT++))
     ((TOTAL_COUNT++))
   fi

   # AWSPrivateLink check
   check_scrubbing "$AWSPL_ERROR" "AWSPrivateLink" && ((PASS_COUNT++))
   ((TOTAL_COUNT++))

   # PrivateLink check
   check_scrubbing "$PL_ERROR" "PrivateLink" && ((PASS_COUNT++))
   ((TOTAL_COUNT++))

   echo -e "\n=== Overall Result ==="
   echo "Controllers passing: $PASS_COUNT / $TOTAL_COUNT"

   if [ $PASS_COUNT -eq $TOTAL_COUNT ]; then
     echo "PASS: All controllers use consistent error scrubbing"
   else
     echo "FAIL: Inconsistent error scrubbing across controllers"
   fi
   ```
   **Expected:**
   - All controllers that have error messages pass UUID absence check
   - 100% consistency in scrubbing behavior

7. **Action:** Cleanup test resources
   ```bash
   oc delete clusterdeployment test-privatelink-scrubbing -n <namespace>
   oc delete secret limited-aws-creds -n <namespace>
   ```
   **Expected:** Resources deleted successfully
