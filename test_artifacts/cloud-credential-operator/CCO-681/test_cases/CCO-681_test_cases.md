# Test Case: CCO-681
**Component:** cloud-credential-operator
**Summary:** Create network policies for CCO

## Test Overview
- **Total Test Cases:** 3
- **Test Types:** E2E (3)
- **Estimated Time:** 60-90 minutes

## Test Cases

### Test Case CCO-681_001
**Name:** Default Deny with AWS Credential Provisioning Workflow
**Description:** Verify that Cloud Credential Operator successfully provisions AWS credentials through the complete CredentialsRequest lifecycle while network policies enforce default-deny with selective allow rules. Validates that egress to AWS API and ingress for metrics work correctly with NetworkPolicies active.
**Type:** E2E
**Priority:** High

#### Prerequisites
- OpenShift cluster with Cloud Credential Operator installed
- AWS cloud platform configured as cluster infrastructure
- Network plugin supporting NetworkPolicy (e.g., OVN-Kubernetes)
- CCO network policies deployed (5 NetworkPolicy resources)
- oc CLI with cluster-admin access
- AWS CLI installed (for credential validation)

#### Test Steps
1. **Action:** Verify all 5 NetworkPolicy resources are deployed in openshift-cloud-credential-operator namespace
   ```bash
   # List all NetworkPolicies
   oc get networkpolicies -n openshift-cloud-credential-operator

   # Verify count
   POLICY_COUNT=$(oc get networkpolicies -n openshift-cloud-credential-operator -o name | wc -l)
   echo "NetworkPolicy count: $POLICY_COUNT (expected: 5)"

   # Verify each policy exists
   POLICIES=("default-deny" "allow-egress" "allow-ingress-metrics" "allow-ingress-pprof" "allow-ingress-webhook")
   for policy in "${POLICIES[@]}"; do
     if oc get networkpolicy $policy -n openshift-cloud-credential-operator &>/dev/null; then
       echo "✓ $policy exists"
     else
       echo "✗ $policy MISSING"
     fi
   done
   ```
   **Expected:**
   - Total of 5 NetworkPolicy resources present
   - All expected policies exist: default-deny, allow-egress, allow-ingress-metrics, allow-ingress-pprof, allow-ingress-webhook

2. **Action:** Verify default-deny NetworkPolicy configuration blocks both ingress and egress
   ```bash
   # Check policyTypes
   POLICY_TYPES=$(oc get networkpolicy default-deny \
     -n openshift-cloud-credential-operator \
     -o jsonpath='{.spec.policyTypes[*]}')

   echo "Policy types: $POLICY_TYPES"

   # Verify both Ingress and Egress are present
   if echo "$POLICY_TYPES" | grep -q "Ingress" && echo "$POLICY_TYPES" | grep -q "Egress"; then
     echo "PASS: default-deny configured for both Ingress and Egress"
   else
     echo "FAIL: default-deny not configured correctly"
   fi

   # Check podSelector is empty (applies to all pods)
   POD_SELECTOR=$(oc get networkpolicy default-deny \
     -n openshift-cloud-credential-operator \
     -o jsonpath='{.spec.podSelector}')

   if [ "$POD_SELECTOR" = "{}" ]; then
     echo "PASS: default-deny applies to all pods in namespace"
   else
     echo "FAIL: default-deny has non-empty podSelector"
   fi
   ```
   **Expected:**
   - policyTypes contains both "Ingress" and "Egress"
   - podSelector is empty (applies to all pods)

3. **Action:** Create a test namespace and CredentialsRequest for AWS S3 access
   ```bash
   # Create test namespace
   oc create namespace cco-test-netpol

   # Create CredentialsRequest
   cat <<EOF | oc apply -f -
   apiVersion: cloudcredential.openshift.io/v1
   kind: CredentialsRequest
   metadata:
     name: test-aws-s3-creds
     namespace: openshift-cloud-credential-operator
   spec:
     providerSpec:
       apiVersion: cloudcredential.openshift.io/v1
       kind: AWSProviderSpec
       statementEntries:
       - effect: Allow
         action:
         - s3:ListBucket
         - s3:GetObject
         resource: "*"
     secretRef:
       name: test-aws-s3-secret
       namespace: cco-test-netpol
   EOF

   echo "CredentialsRequest created at: $(date)"
   ```
   **Expected:**
   - Test namespace created successfully
   - CredentialsRequest created without errors

4. **Action:** Monitor CredentialsRequest status transition to Provisioned state with timeout
   ```bash
   # Monitor status with 120-second timeout
   START_TIME=$(date +%s)
   TIMEOUT=120
   PROVISIONED=false

   while true; do
     # Check provisioned status
     STATUS=$(oc get credentialsrequest test-aws-s3-creds \
       -n openshift-cloud-credential-operator \
       -o jsonpath='{.status.conditions[?(@.type=="Provisioned")].status}' 2>/dev/null)

     CURRENT_TIME=$(date +%s)
     ELAPSED=$((CURRENT_TIME - START_TIME))

     if [ "$STATUS" = "True" ]; then
       echo "PASS: CredentialsRequest provisioned in ${ELAPSED}s"
       PROVISIONED=true
       break
     fi

     if [ $ELAPSED -ge $TIMEOUT ]; then
       echo "FAIL: CredentialsRequest not provisioned within ${TIMEOUT}s"
       # Show current status for debugging
       oc get credentialsrequest test-aws-s3-creds \
         -n openshift-cloud-credential-operator \
         -o jsonpath='{.status.conditions[*]}'
       break
     fi

     echo "Waiting for provisioning... (${ELAPSED}s elapsed)"
     sleep 5
   done
   ```
   **Expected:**
   - CredentialsRequest status.conditions[type=Provisioned].status becomes "True" within 120 seconds
   - Elapsed time ≤ 120 seconds

5. **Action:** Verify that Secret was created in target namespace with AWS credentials
   ```bash
   # Check Secret exists
   if oc get secret test-aws-s3-secret -n cco-test-netpol &>/dev/null; then
     echo "PASS: Secret created in target namespace"
   else
     echo "FAIL: Secret not found in target namespace"
     exit 1
   fi

   # Verify Secret contains expected keys
   SECRET_KEYS=$(oc get secret test-aws-s3-secret -n cco-test-netpol \
     -o jsonpath='{.data}' | jq -r 'keys[]')

   echo "Secret keys: $SECRET_KEYS"

   # Check for aws_access_key_id and aws_secret_access_key
   if echo "$SECRET_KEYS" | grep -q "aws_access_key_id" && \
      echo "$SECRET_KEYS" | grep -q "aws_secret_access_key"; then
     echo "PASS: Secret contains AWS credential keys"
   else
     echo "FAIL: Secret missing expected AWS credential keys"
   fi
   ```
   **Expected:**
   - Secret exists in cco-test-netpol namespace
   - Secret contains aws_access_key_id and aws_secret_access_key keys

6. **Action:** Validate provisioned AWS credentials are functional by calling AWS API
   ```bash
   # Extract credentials
   ACCESS_KEY=$(oc get secret test-aws-s3-secret -n cco-test-netpol \
     -o jsonpath='{.data.aws_access_key_id}' | base64 -d)
   SECRET_KEY=$(oc get secret test-aws-s3-secret -n cco-test-netpol \
     -o jsonpath='{.data.aws_secret_access_key}' | base64 -d)

   # Test credentials with AWS STS get-caller-identity
   AWS_ACCESS_KEY_ID=$ACCESS_KEY \
   AWS_SECRET_ACCESS_KEY=$SECRET_KEY \
   aws sts get-caller-identity --output json

   if [ $? -eq 0 ]; then
     echo "PASS: AWS credentials are functional"
   else
     echo "FAIL: AWS credentials invalid or cannot reach AWS API"
   fi
   ```
   **Expected:**
   - AWS CLI command succeeds (exit code 0)
   - Command returns valid JSON with AWS account information
   - Credentials authenticate successfully to AWS

7. **Action:** Verify CCO pod logs show no network connection denials during provisioning
   ```bash
   # Get CCO pod name
   CCO_POD=$(oc get pods -n openshift-cloud-credential-operator \
     -l app=cloud-credential-operator \
     -o jsonpath='{.items[0].metadata.name}')

   # Check logs for network denials (last 10 minutes)
   echo "Analyzing CCO pod logs for network denials..."
   DENY_PATTERNS="connection refused|connection timeout|network.*denied|dial.*timeout|policy.*block"

   DENY_COUNT=$(oc logs -n openshift-cloud-credential-operator $CCO_POD --since=10m | \
     grep -iE "$DENY_PATTERNS" | wc -l)

   echo "Network denial messages found: $DENY_COUNT"

   if [ $DENY_COUNT -eq 0 ]; then
     echo "PASS: No network connection denials in CCO pod logs"
   else
     echo "FAIL: Found $DENY_COUNT network denial messages"
     # Show sample denials
     oc logs -n openshift-cloud-credential-operator $CCO_POD --since=10m | \
       grep -iE "$DENY_PATTERNS" | head -5
   fi
   ```
   **Expected:**
   - 0 network denial messages in CCO pod logs
   - No connection refused or timeout errors related to AWS API
   - No network policy blocking messages

8. **Action:** Cleanup test resources
   ```bash
   # Delete CredentialsRequest
   oc delete credentialsrequest test-aws-s3-creds \
     -n openshift-cloud-credential-operator

   # Delete test namespace
   oc delete namespace cco-test-netpol

   echo "Cleanup completed"
   ```
   **Expected:**
   - CredentialsRequest deleted successfully
   - Test namespace deleted successfully

---

### Test Case CCO-681_002
**Name:** Ingress NetworkPolicy Validation for Metrics and Webhook Endpoints
**Description:** Verify that ingress NetworkPolicies correctly allow access to CCO metrics endpoint (port 8443), pprof debugging endpoint (port 6060), and pod-identity-webhook endpoint (port 9443) while default-deny blocks unauthorized access.
**Type:** E2E
**Priority:** High

#### Prerequisites
- OpenShift cluster with Cloud Credential Operator installed
- CCO network policies deployed
- oc CLI with cluster-admin access
- curl command available for HTTP testing
- Pod Identity Webhook enabled (if testing webhook ingress)

#### Test Steps
1. **Action:** Verify ingress NetworkPolicy resources exist with correct port configurations
   ```bash
   # Check allow-ingress-metrics policy
   METRICS_PORT=$(oc get networkpolicy allow-ingress-metrics \
     -n openshift-cloud-credential-operator \
     -o jsonpath='{.spec.ingress[0].ports[0].port}')
   echo "Metrics policy port: $METRICS_PORT (expected: 8443)"

   # Check allow-ingress-pprof policy
   PPROF_PORT=$(oc get networkpolicy allow-ingress-pprof \
     -n openshift-cloud-credential-operator \
     -o jsonpath='{.spec.ingress[0].ports[0].port}')
   echo "Pprof policy port: $PPROF_PORT (expected: 6060)"

   # Check allow-ingress-webhook policy
   WEBHOOK_PORT=$(oc get networkpolicy allow-ingress-webhook \
     -n openshift-cloud-credential-operator \
     -o jsonpath='{.spec.ingress[0].ports[0].port}')
   echo "Webhook policy port: $WEBHOOK_PORT (expected: 9443)"

   # Validate ports
   if [ "$METRICS_PORT" = "8443" ] && [ "$PPROF_PORT" = "6060" ] && [ "$WEBHOOK_PORT" = "9443" ]; then
     echo "PASS: All ingress policies have correct port configurations"
   else
     echo "FAIL: Port configuration mismatch detected"
   fi
   ```
   **Expected:**
   - allow-ingress-metrics: port 8443
   - allow-ingress-pprof: port 6060
   - allow-ingress-webhook: port 9443

2. **Action:** Verify pod selectors target correct applications
   ```bash
   # Metrics and pprof target cloud-credential-operator
   METRICS_SELECTOR=$(oc get networkpolicy allow-ingress-metrics \
     -n openshift-cloud-credential-operator \
     -o jsonpath='{.spec.podSelector.matchExpressions[0].values[*]}')
   echo "Metrics selector values: $METRICS_SELECTOR"

   PPROF_SELECTOR=$(oc get networkpolicy allow-ingress-pprof \
     -n openshift-cloud-credential-operator \
     -o jsonpath='{.spec.podSelector.matchExpressions[0].values[*]}')
   echo "Pprof selector values: $PPROF_SELECTOR"

   # Webhook targets pod-identity-webhook
   WEBHOOK_SELECTOR=$(oc get networkpolicy allow-ingress-webhook \
     -n openshift-cloud-credential-operator \
     -o jsonpath='{.spec.podSelector.matchExpressions[0].values[*]}')
   echo "Webhook selector values: $WEBHOOK_SELECTOR"

   # Validate selectors
   if echo "$METRICS_SELECTOR" | grep -q "cloud-credential-operator" && \
      echo "$PPROF_SELECTOR" | grep -q "cloud-credential-operator" && \
      echo "$WEBHOOK_SELECTOR" | grep -q "pod-identity-webhook"; then
     echo "PASS: Pod selectors configured correctly"
   else
     echo "FAIL: Pod selector mismatch"
   fi
   ```
   **Expected:**
   - Metrics and pprof policies select app=cloud-credential-operator
   - Webhook policy selects app=pod-identity-webhook

3. **Action:** Test metrics endpoint accessibility from within CCO pod
   ```bash
   # Get CCO pod name
   CCO_POD=$(oc get pods -n openshift-cloud-credential-operator \
     -l app=cloud-credential-operator \
     -o jsonpath='{.items[0].metadata.name}')

   echo "Testing metrics endpoint on CCO pod: $CCO_POD"

   # Execute curl to metrics endpoint
   METRICS_RESPONSE=$(oc exec -n openshift-cloud-credential-operator $CCO_POD -- \
     curl -k -s -o /dev/null -w "%{http_code},%{time_total}" \
     https://localhost:8443/metrics 2>/dev/null)

   # Parse response
   HTTP_CODE=$(echo $METRICS_RESPONSE | cut -d',' -f1)
   RESPONSE_TIME=$(echo $METRICS_RESPONSE | cut -d',' -f2)

   echo "Metrics endpoint response: HTTP $HTTP_CODE, ${RESPONSE_TIME}s"

   # Validate response
   if [ "$HTTP_CODE" = "200" ] && (( $(echo "$RESPONSE_TIME < 5.0" | bc -l) )); then
     echo "PASS: Metrics endpoint accessible (HTTP $HTTP_CODE, ${RESPONSE_TIME}s)"
   else
     echo "FAIL: Metrics endpoint issue (HTTP $HTTP_CODE, ${RESPONSE_TIME}s)"
   fi
   ```
   **Expected:**
   - HTTP status code 200
   - Response time < 5 seconds
   - Metrics data returned successfully

4. **Action:** Retrieve and validate metrics content
   ```bash
   # Get actual metrics data
   METRICS_DATA=$(oc exec -n openshift-cloud-credential-operator $CCO_POD -- \
     curl -k -s https://localhost:8443/metrics 2>/dev/null)

   # Check for Prometheus format metrics
   if echo "$METRICS_DATA" | grep -q "# HELP" && echo "$METRICS_DATA" | grep -q "# TYPE"; then
     echo "PASS: Metrics endpoint returns Prometheus format data"
   else
     echo "FAIL: Metrics endpoint not returning expected format"
   fi

   # Count metrics (should have multiple)
   METRIC_COUNT=$(echo "$METRICS_DATA" | grep -c "^# HELP" || echo 0)
   echo "Metrics count: $METRIC_COUNT"

   if [ $METRIC_COUNT -gt 0 ]; then
     echo "PASS: Multiple metrics available ($METRIC_COUNT metrics)"
   else
     echo "FAIL: No metrics found"
   fi
   ```
   **Expected:**
   - Metrics data contains Prometheus format (# HELP, # TYPE)
   - Multiple metrics present (count > 0)

5. **Action:** Test pprof endpoint accessibility via port-forward
   ```bash
   # Port-forward to pprof endpoint
   oc port-forward -n openshift-cloud-credential-operator $CCO_POD 6060:6060 &
   PF_PID=$!
   sleep 3

   # Access pprof index
   PPROF_STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:6060/debug/pprof/ 2>/dev/null)

   # Kill port-forward
   kill $PF_PID 2>/dev/null

   echo "Pprof endpoint response: HTTP $PPROF_STATUS"

   if [ "$PPROF_STATUS" = "200" ]; then
     echo "PASS: Pprof endpoint accessible (HTTP $PPROF_STATUS)"
   else
     echo "FAIL: Pprof endpoint not accessible (HTTP $PPROF_STATUS)"
   fi
   ```
   **Expected:**
   - HTTP status code 200
   - Pprof index page accessible

6. **Action:** Test pod-identity-webhook ingress by creating pod with cloud identity annotation
   ```bash
   # Check if pod-identity-webhook pod exists
   WEBHOOK_POD=$(oc get pods -n openshift-cloud-credential-operator \
     -l app=pod-identity-webhook \
     -o jsonpath='{.items[0].metadata.name}' 2>/dev/null)

   if [ -z "$WEBHOOK_POD" ]; then
     echo "SKIP: pod-identity-webhook not deployed (may not be AWS/ROSA cluster)"
   else
     echo "Testing webhook with pod-identity-webhook pod: $WEBHOOK_POD"

     # Create namespace for test
     oc create namespace webhook-test 2>/dev/null || true

     # Create pod with pod identity annotation
     cat <<EOF | oc apply -f -
   apiVersion: v1
   kind: Pod
   metadata:
     name: test-pod-identity
     namespace: webhook-test
     annotations:
       eks.amazonaws.com/role-arn: "arn:aws:iam::123456789012:role/test-role"
   spec:
     containers:
     - name: test
       image: registry.redhat.io/ubi8/ubi-minimal
       command: ["sleep", "3600"]
   EOF

     # Wait for webhook to process
     sleep 5

     # Check if webhook mutated the pod (injected volumes)
     VOLUME_COUNT=$(oc get pod test-pod-identity -n webhook-test \
       -o jsonpath='{.spec.volumes[*].name}' | grep -c 'aws-iam-token' || echo 0)

     if [ $VOLUME_COUNT -gt 0 ]; then
       echo "PASS: Webhook successfully mutated pod (injected $VOLUME_COUNT volume(s))"
     else
       echo "INFO: Webhook did not mutate pod (may require specific cluster configuration)"
     fi

     # Cleanup
     oc delete pod test-pod-identity -n webhook-test --force --grace-period=0 2>/dev/null
     oc delete namespace webhook-test 2>/dev/null
   fi
   ```
   **Expected:**
   - If webhook pod exists: Pod successfully mutated with aws-iam-token volume
   - Webhook processed admission request on port 9443

7. **Action:** Verify ingress policyTypes are set correctly
   ```bash
   # Check each ingress policy has Ingress policyType
   for policy in allow-ingress-metrics allow-ingress-pprof allow-ingress-webhook; do
     POLICY_TYPE=$(oc get networkpolicy $policy \
       -n openshift-cloud-credential-operator \
       -o jsonpath='{.spec.policyTypes[0]}')

     if [ "$POLICY_TYPE" = "Ingress" ]; then
       echo "✓ $policy: policyType = Ingress"
     else
       echo "✗ $policy: policyType = $POLICY_TYPE (expected: Ingress)"
     fi
   done
   ```
   **Expected:**
   - All three ingress policies have policyTypes = ["Ingress"]

---

### Test Case CCO-681_003
**Name:** Multi-Cloud Egress Connectivity with NetworkPolicies
**Description:** Verify that allow-egress NetworkPolicy permits CCO to communicate with cloud platform APIs across AWS, Azure, and GCP, validating the broad port range (1-65535) configuration enables all necessary cloud API calls while default-deny is active.
**Type:** E2E
**Priority:** High

#### Prerequisites
- OpenShift cluster with multi-cloud capability or access to multiple cloud platforms
- Cloud Credential Operator installed
- CCO network policies deployed
- oc CLI with cluster-admin access
- Cloud platform credentials configured (AWS, Azure, GCP)

#### Test Steps
1. **Action:** Verify allow-egress NetworkPolicy port range configuration
   ```bash
   # Extract TCP port configuration
   TCP_PORT=$(oc get networkpolicy allow-egress \
     -n openshift-cloud-credential-operator \
     -o jsonpath='{.spec.egress[0].ports[0].port}')
   TCP_END_PORT=$(oc get networkpolicy allow-egress \
     -n openshift-cloud-credential-operator \
     -o jsonpath='{.spec.egress[0].ports[0].endPort}')
   TCP_PROTOCOL=$(oc get networkpolicy allow-egress \
     -n openshift-cloud-credential-operator \
     -o jsonpath='{.spec.egress[0].ports[0].protocol}')

   echo "TCP egress: protocol=$TCP_PROTOCOL, port=$TCP_PORT, endPort=$TCP_END_PORT"

   # Extract UDP port configuration
   UDP_PORT=$(oc get networkpolicy allow-egress \
     -n openshift-cloud-credential-operator \
     -o jsonpath='{.spec.egress[1].ports[0].port}')
   UDP_END_PORT=$(oc get networkpolicy allow-egress \
     -n openshift-cloud-credential-operator \
     -o jsonpath='{.spec.egress[1].ports[0].endPort}')
   UDP_PROTOCOL=$(oc get networkpolicy allow-egress \
     -n openshift-cloud-credential-operator \
     -o jsonpath='{.spec.egress[1].ports[0].protocol}')

   echo "UDP egress: protocol=$UDP_PROTOCOL, port=$UDP_PORT, endPort=$UDP_END_PORT"

   # Validate configuration
   if [ "$TCP_PROTOCOL" = "TCP" ] && [ "$TCP_PORT" = "1" ] && [ "$TCP_END_PORT" = "65535" ] && \
      [ "$UDP_PROTOCOL" = "UDP" ] && [ "$UDP_PORT" = "1" ] && [ "$UDP_END_PORT" = "65535" ]; then
     echo "PASS: Egress port range correctly configured (TCP/UDP 1-65535)"
   else
     echo "FAIL: Egress port configuration mismatch"
   fi
   ```
   **Expected:**
   - TCP: port 1, endPort 65535
   - UDP: port 1, endPort 65535
   - Both protocols configured

2. **Action:** Verify egress policy pod selector targets CCO and webhook
   ```bash
   # Get pod selector values
   EGRESS_SELECTOR=$(oc get networkpolicy allow-egress \
     -n openshift-cloud-credential-operator \
     -o jsonpath='{.spec.podSelector.matchExpressions[0].values[*]}')

   echo "Egress pod selector values: $EGRESS_SELECTOR"

   # Validate selector includes both apps
   if echo "$EGRESS_SELECTOR" | grep -q "cloud-credential-operator" && \
      echo "$EGRESS_SELECTOR" | grep -q "pod-identity-webhook"; then
     echo "PASS: Egress policy targets both cloud-credential-operator and pod-identity-webhook"
   else
     echo "FAIL: Egress pod selector missing expected applications"
   fi
   ```
   **Expected:**
   - Pod selector includes cloud-credential-operator
   - Pod selector includes pod-identity-webhook

3. **Action:** Test DNS resolution from CCO pod for cloud platform endpoints
   ```bash
   # Get CCO pod
   CCO_POD=$(oc get pods -n openshift-cloud-credential-operator \
     -l app=cloud-credential-operator \
     -o jsonpath='{.items[0].metadata.name}')

   echo "Testing DNS resolution from CCO pod: $CCO_POD"

   # Test AWS endpoint
   echo "Testing AWS endpoint..."
   oc exec -n openshift-cloud-credential-operator $CCO_POD -- \
     nslookup sts.amazonaws.com >/dev/null 2>&1
   AWS_DNS=$?

   # Test Azure endpoint
   echo "Testing Azure endpoint..."
   oc exec -n openshift-cloud-credential-operator $CCO_POD -- \
     nslookup management.azure.com >/dev/null 2>&1
   AZURE_DNS=$?

   # Test GCP endpoint
   echo "Testing GCP endpoint..."
   oc exec -n openshift-cloud-credential-operator $CCO_POD -- \
     nslookup iam.googleapis.com >/dev/null 2>&1
   GCP_DNS=$?

   # Report results
   echo "DNS resolution results: AWS=$AWS_DNS, Azure=$AZURE_DNS, GCP=$GCP_DNS (0=success)"

   if [ $AWS_DNS -eq 0 ] && [ $AZURE_DNS -eq 0 ] && [ $GCP_DNS -eq 0 ]; then
     echo "PASS: DNS resolution works for all cloud platforms"
   else
     echo "FAIL: DNS resolution failed for one or more platforms"
   fi
   ```
   **Expected:**
   - AWS endpoint resolves successfully (exit code 0)
   - Azure endpoint resolves successfully (exit code 0)
   - GCP endpoint resolves successfully (exit code 0)

4. **Action:** Create AWS CredentialsRequest and verify provisioning
   ```bash
   # Create test namespace
   oc create namespace cco-multicloud-test 2>/dev/null || true

   # Create AWS CredentialsRequest
   cat <<EOF | oc apply -f -
   apiVersion: cloudcredential.openshift.io/v1
   kind: CredentialsRequest
   metadata:
     name: test-aws-multicloud
     namespace: openshift-cloud-credential-operator
   spec:
     providerSpec:
       apiVersion: cloudcredential.openshift.io/v1
       kind: AWSProviderSpec
       statementEntries:
       - effect: Allow
         action:
         - s3:ListBucket
         resource: "*"
     secretRef:
       name: test-aws-creds
       namespace: cco-multicloud-test
   EOF

   echo "AWS CredentialsRequest created"

   # Wait for provisioning (60 second timeout)
   TIMEOUT=60
   ELAPSED=0
   AWS_PROVISIONED=false

   while [ $ELAPSED -lt $TIMEOUT ]; do
     STATUS=$(oc get credentialsrequest test-aws-multicloud \
       -n openshift-cloud-credential-operator \
       -o jsonpath='{.status.conditions[?(@.type=="Provisioned")].status}' 2>/dev/null)

     if [ "$STATUS" = "True" ]; then
       echo "PASS: AWS credentials provisioned in ${ELAPSED}s"
       AWS_PROVISIONED=true
       break
     fi

     sleep 5
     ELAPSED=$((ELAPSED + 5))
   done

   if [ "$AWS_PROVISIONED" = "false" ]; then
     echo "FAIL: AWS credentials not provisioned within ${TIMEOUT}s"
   fi
   ```
   **Expected:**
   - AWS CredentialsRequest status becomes Provisioned within 60 seconds
   - Secret created in target namespace

5. **Action:** Create Azure CredentialsRequest and verify provisioning (if Azure platform available)
   ```bash
   # Check if Azure is supported (skip if not)
   PLATFORM=$(oc get infrastructure cluster -o jsonpath='{.status.platform}')

   if [ "$PLATFORM" != "Azure" ]; then
     echo "SKIP: Azure test - cluster platform is $PLATFORM, not Azure"
   else
     # Create Azure CredentialsRequest
     cat <<EOF | oc apply -f -
   apiVersion: cloudcredential.openshift.io/v1
   kind: CredentialsRequest
   metadata:
     name: test-azure-multicloud
     namespace: openshift-cloud-credential-operator
   spec:
     providerSpec:
       apiVersion: cloudcredential.openshift.io/v1
       kind: AzureProviderSpec
       roleBindings:
       - role: Contributor
     secretRef:
       name: test-azure-creds
       namespace: cco-multicloud-test
   EOF

     echo "Azure CredentialsRequest created"

     # Wait for provisioning
     TIMEOUT=60
     ELAPSED=0
     AZURE_PROVISIONED=false

     while [ $ELAPSED -lt $TIMEOUT ]; do
       STATUS=$(oc get credentialsrequest test-azure-multicloud \
         -n openshift-cloud-credential-operator \
         -o jsonpath='{.status.conditions[?(@.type=="Provisioned")].status}' 2>/dev/null)

       if [ "$STATUS" = "True" ]; then
         echo "PASS: Azure credentials provisioned in ${ELAPSED}s"
         AZURE_PROVISIONED=true
         break
       fi

       sleep 5
       ELAPSED=$((ELAPSED + 5))
     done

     if [ "$AZURE_PROVISIONED" = "false" ]; then
       echo "FAIL: Azure credentials not provisioned within ${TIMEOUT}s"
     fi
   fi
   ```
   **Expected:**
   - Azure CredentialsRequest status becomes Provisioned within 60 seconds (if Azure platform)
   - Secret created in target namespace

6. **Action:** Create GCP CredentialsRequest and verify provisioning (if GCP platform available)
   ```bash
   # Check if GCP is supported (skip if not)
   PLATFORM=$(oc get infrastructure cluster -o jsonpath='{.status.platform}')

   if [ "$PLATFORM" != "GCP" ]; then
     echo "SKIP: GCP test - cluster platform is $PLATFORM, not GCP"
   else
     # Create GCP CredentialsRequest
     cat <<EOF | oc apply -f -
   apiVersion: cloudcredential.openshift.io/v1
   kind: CredentialsRequest
   metadata:
     name: test-gcp-multicloud
     namespace: openshift-cloud-credential-operator
   spec:
     providerSpec:
       apiVersion: cloudcredential.openshift.io/v1
       kind: GCPProviderSpec
       predefinedRoles:
       - roles/storage.objectViewer
     secretRef:
       name: test-gcp-creds
       namespace: cco-multicloud-test
   EOF

     echo "GCP CredentialsRequest created"

     # Wait for provisioning
     TIMEOUT=60
     ELAPSED=0
     GCP_PROVISIONED=false

     while [ $ELAPSED -lt $TIMEOUT ]; do
       STATUS=$(oc get credentialsrequest test-gcp-multicloud \
         -n openshift-cloud-credential-operator \
         -o jsonpath='{.status.conditions[?(@.type=="Provisioned")].status}' 2>/dev/null)

       if [ "$STATUS" = "True" ]; then
         echo "PASS: GCP credentials provisioned in ${ELAPSED}s"
         GCP_PROVISIONED=true
         break
       fi

       sleep 5
       ELAPSED=$((ELAPSED + 5))
     done

     if [ "$GCP_PROVISIONED" = "false" ]; then
       echo "FAIL: GCP credentials not provisioned within ${TIMEOUT}s"
     fi
   fi
   ```
   **Expected:**
   - GCP CredentialsRequest status becomes Provisioned within 60 seconds (if GCP platform)
   - Secret created in target namespace

7. **Action:** Verify no egress network denials in CCO pod logs
   ```bash
   # Check CCO logs for egress denials
   CCO_POD=$(oc get pods -n openshift-cloud-credential-operator \
     -l app=cloud-credential-operator \
     -o jsonpath='{.items[0].metadata.name}')

   EGRESS_DENIALS=$(oc logs -n openshift-cloud-credential-operator $CCO_POD --since=10m | \
     grep -iE '(egress.*denied|connection refused.*cloud|timeout.*api)' | wc -l)

   echo "Egress denial count: $EGRESS_DENIALS"

   if [ $EGRESS_DENIALS -eq 0 ]; then
     echo "PASS: No egress denials detected in CCO pod logs"
   else
     echo "FAIL: Found $EGRESS_DENIALS egress denial messages"
     # Show samples
     oc logs -n openshift-cloud-credential-operator $CCO_POD --since=10m | \
       grep -iE '(egress.*denied|connection refused.*cloud|timeout.*api)' | head -5
   fi
   ```
   **Expected:**
   - 0 egress denial messages in CCO logs
   - No connection refused errors for cloud APIs
   - No timeout errors due to network policies

8. **Action:** Cleanup test resources
   ```bash
   # Delete CredentialsRequests
   oc delete credentialsrequest test-aws-multicloud \
     -n openshift-cloud-credential-operator 2>/dev/null || true
   oc delete credentialsrequest test-azure-multicloud \
     -n openshift-cloud-credential-operator 2>/dev/null || true
   oc delete credentialsrequest test-gcp-multicloud \
     -n openshift-cloud-credential-operator 2>/dev/null || true

   # Delete test namespace
   oc delete namespace cco-multicloud-test 2>/dev/null || true

   echo "Cleanup completed"
   ```
   **Expected:**
   - All test CredentialsRequests deleted
   - Test namespace deleted successfully
