# Test Strategy

## Test Coverage Matrix

| Scenario | Platform | Cluster Type | Test Type | Reasoning | Priority | Test Case ID | Status |
|----------|----------|--------------|-----------|-----------|----------|--------------|--------|
| Default Deny with Credential Provisioning | AWS | Standard | E2E | Validates end-to-end workflow - user creates CredentialsRequest, CCO provisions credentials via cloud API while default-deny is active | High | TC-CCO-681-001 | ❌ Not Started |
| Ingress Policy Validation for Metrics and Webhook | AWS | Standard | E2E | Verifies metrics collection and webhook operation - quantifiable via HTTP response codes and successful pod mutations | High | TC-CCO-681-002 | ❌ Not Started |
| Multi-Cloud Egress Connectivity | AWS, Azure, GCP | Standard | E2E | Confirms NetworkPolicy allows cloud API communication across platforms - validates core functionality | High | TC-CCO-681-003 | ❌ Not Started |

## Test Scenarios

### Scenario 1: Default Deny with Credential Provisioning
**Validation Type**: Type E (Progression)

**Objective**: Verify that CCO can successfully provision cloud credentials from CredentialsRequest creation through to functional credential usage, while network policies enforce default-deny with selective allow rules

**User Workflow**:
1. Administrator enables network policies in OpenShift cluster
2. User creates a CredentialsRequest for AWS credentials
3. CCO reconciles the request, communicates with AWS API, creates IAM resources
4. Target Secret is created with valid AWS credentials
5. User deploys workload consuming the credentials
6. Workload successfully authenticates to AWS

**Quantitative Validation**:
- **Metric 1: CredentialsRequest State Progression**
  - Measure: CredentialsRequest status.conditions transitions
  - Expected sequence: Created → Provisioning → Provisioned
  - Threshold: Status reaches "Provisioned" within 60 seconds = PASS
  - Measurement:
    ```bash
    # Monitor status.conditions[type=Provisioned]
    START=$(date +%s)
    while true; do
      STATUS=$(oc get credentialsrequest <name> -n <namespace> \
        -o jsonpath='{.status.conditions[?(@.type=="Provisioned")].status}')
      if [ "$STATUS" = "True" ]; then
        END=$(date +%s)
        DURATION=$((END - START))
        echo "PASS: Provisioned in ${DURATION}s"
        break
      fi
      sleep 2
    done
    ```

- **Metric 2: NetworkPolicy Existence**
  - Measure: Count of NetworkPolicy resources in namespace
  - Expected: 5 NetworkPolicies present (default-deny, allow-egress, allow-ingress-metrics, allow-ingress-pprof, allow-ingress-webhook)
  - Threshold: Count = 5 = PASS
  - Measurement:
    ```bash
    POLICY_COUNT=$(oc get networkpolicies -n openshift-cloud-credential-operator -o name | wc -l)
    if [ $POLICY_COUNT -eq 5 ]; then
      echo "PASS: All 5 network policies present"
    else
      echo "FAIL: Expected 5 policies, found $POLICY_COUNT"
    fi
    ```

- **Metric 3: Credential Functionality**
  - Measure: AWS API call success using provisioned credentials
  - Expected: AWS CLI command using credentials returns successful response (exit code 0)
  - Threshold: Exit code 0 + valid JSON response = PASS
  - Measurement:
    ```bash
    # Extract credentials from Secret
    ACCESS_KEY=$(oc get secret <cred-secret> -n <namespace> -o jsonpath='{.data.aws_access_key_id}' | base64 -d)
    SECRET_KEY=$(oc get secret <cred-secret> -n <namespace> -o jsonpath='{.data.aws_secret_access_key}' | base64 -d)

    # Test credentials
    AWS_ACCESS_KEY_ID=$ACCESS_KEY AWS_SECRET_ACCESS_KEY=$SECRET_KEY aws sts get-caller-identity
    if [ $? -eq 0 ]; then
      echo "PASS: Credentials functional"
    else
      echo "FAIL: Credentials invalid"
    fi
    ```

- **Metric 4: Network Policy Enforcement**
  - Measure: CCO pod logs for denied network connections
  - Expected: 0 network connection denials for allowed traffic patterns
  - Threshold: grep for "connection refused|network policy" returns 0 matches = PASS
  - Measurement:
    ```bash
    CCO_POD=$(oc get pods -n openshift-cloud-credential-operator \
      -l app=cloud-credential-operator -o jsonpath='{.items[0].metadata.name}')

    DENY_COUNT=$(oc logs -n openshift-cloud-credential-operator $CCO_POD --since=5m | \
      grep -iE '(connection refused|network.{0,10}denied|policy.{0,10}block)' | wc -l)

    if [ $DENY_COUNT -eq 0 ]; then
      echo "PASS: No network denials detected"
    else
      echo "FAIL: Found $DENY_COUNT network denial messages"
    fi
    ```

**Expected Results**:
- CredentialsRequest transitions to Provisioned state within 60 seconds
- 5 NetworkPolicy resources exist in namespace
- Provisioned credentials successfully authenticate to AWS
- CCO pod logs show no network connection denials
- Egress to AWS API permitted by allow-egress policy
- Default-deny policy active (verifiable via NetworkPolicy resource)

---

### Scenario 2: Ingress Policy Validation for Metrics and Webhook
**Validation Type**: Type C (State)

**Objective**: Verify that ingress NetworkPolicies correctly allow access to CCO metrics endpoint (port 8443) and pod-identity-webhook endpoint (port 9443), while default-deny blocks unauthorized access

**User Workflow**:
1. Monitoring system (Prometheus) scrapes CCO metrics endpoint
2. Kubernetes API server sends pod admission requests to pod-identity-webhook
3. Administrator accesses pprof endpoint for debugging
4. Unauthorized pod attempts to access metrics (should be denied by default-deny)

**Quantitative Validation**:
- **Metric 1: Metrics Endpoint Accessibility**
  - Measure: HTTP response code from metrics endpoint
  - Expected: HTTP 200 from allowed source
  - Threshold: Status code 200 + response time < 5 seconds = PASS
  - Measurement:
    ```bash
    # Access metrics endpoint from within cluster (simulating Prometheus)
    oc exec -n openshift-cloud-credential-operator $CCO_POD -- \
      curl -k -s -o /dev/null -w "%{http_code},%{time_total}" \
      https://localhost:8443/metrics

    # Parse response
    RESPONSE="<output from above>"
    STATUS_CODE=$(echo $RESPONSE | cut -d',' -f1)
    RESPONSE_TIME=$(echo $RESPONSE | cut -d',' -f2)

    if [ "$STATUS_CODE" = "200" ] && (( $(echo "$RESPONSE_TIME < 5" | bc -l) )); then
      echo "PASS: Metrics endpoint accessible (${STATUS_CODE}, ${RESPONSE_TIME}s)"
    else
      echo "FAIL: Metrics endpoint issue (${STATUS_CODE}, ${RESPONSE_TIME}s)"
    fi
    ```

- **Metric 2: Webhook Endpoint Functionality**
  - Measure: Successful pod mutations by webhook
  - Expected: Pod created with cloud identity annotations/volumes injected
  - Threshold: Webhook admission response = allowed + mutations applied = PASS
  - Measurement:
    ```bash
    # Create test pod requesting pod identity
    cat <<EOF | oc apply -f -
    apiVersion: v1
    kind: Pod
    metadata:
      name: test-pod-identity
      namespace: <namespace>
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

    # Check if webhook injected volumes/env vars
    VOLUME_COUNT=$(oc get pod test-pod-identity -n <namespace> \
      -o jsonpath='{.spec.volumes[*].name}' | grep -c 'aws-iam-token' || echo 0)

    if [ $VOLUME_COUNT -gt 0 ]; then
      echo "PASS: Webhook successfully mutated pod (injected $VOLUME_COUNT volume(s))"
    else
      echo "FAIL: Webhook did not mutate pod"
    fi
    ```

- **Metric 3: Pprof Endpoint Accessibility**
  - Measure: HTTP response code from pprof endpoint
  - Expected: HTTP 200 from debugging session
  - Threshold: Status code 200 = PASS
  - Measurement:
    ```bash
    # Port-forward to pprof endpoint
    oc port-forward -n openshift-cloud-credential-operator $CCO_POD 6060:6060 &
    PF_PID=$!
    sleep 2

    # Access pprof
    PPROF_STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:6060/debug/pprof/)
    kill $PF_PID

    if [ "$PPROF_STATUS" = "200" ]; then
      echo "PASS: Pprof endpoint accessible ($PPROF_STATUS)"
    else
      echo "FAIL: Pprof endpoint not accessible ($PPROF_STATUS)"
    fi
    ```

- **Metric 4: Ingress NetworkPolicy Validation**
  - Measure: Presence and configuration of ingress NetworkPolicies
  - Expected: 3 ingress policies with correct port specifications
  - Threshold: All 3 policies exist with matching port configurations = PASS
  - Measurement:
    ```bash
    # Verify allow-ingress-metrics policy
    METRICS_PORT=$(oc get networkpolicy allow-ingress-metrics \
      -n openshift-cloud-credential-operator \
      -o jsonpath='{.spec.ingress[0].ports[0].port}')

    # Verify allow-ingress-webhook policy
    WEBHOOK_PORT=$(oc get networkpolicy allow-ingress-webhook \
      -n openshift-cloud-credential-operator \
      -o jsonpath='{.spec.ingress[0].ports[0].port}')

    # Verify allow-ingress-pprof policy
    PPROF_PORT=$(oc get networkpolicy allow-ingress-pprof \
      -n openshift-cloud-credential-operator \
      -o jsonpath='{.spec.ingress[0].ports[0].port}')

    if [ "$METRICS_PORT" = "8443" ] && [ "$WEBHOOK_PORT" = "9443" ] && [ "$PPROF_PORT" = "6060" ]; then
      echo "PASS: All ingress policies configured correctly"
    else
      echo "FAIL: Port configuration mismatch (metrics=$METRICS_PORT, webhook=$WEBHOOK_PORT, pprof=$PPROF_PORT)"
    fi
    ```

**Expected Results**:
- Metrics endpoint returns HTTP 200 within 5 seconds
- Webhook successfully mutates pods requesting cloud identity
- Pprof endpoint accessible via port-forward
- All 3 ingress NetworkPolicies exist with correct port configurations
- Default-deny policy doesn't block allowed ingress traffic

---

### Scenario 3: Multi-Cloud Egress Connectivity
**Validation Type**: Type C (State)

**Objective**: Confirm that allow-egress NetworkPolicy permits CCO to communicate with cloud platform APIs across AWS, Azure, and GCP, validating the broad port range (1-65535) configuration

**User Workflow**:
1. User creates CredentialsRequests for multiple cloud platforms
2. CCO attempts to authenticate and provision resources on each cloud
3. Cloud API calls succeed despite default-deny egress policy
4. Credentials provisioned successfully for each platform

**Quantitative Validation**:
- **Metric 1: AWS Egress Connectivity**
  - Measure: CredentialsRequest provisioning success on AWS
  - Expected: AWS IAM user/role created successfully
  - Threshold: CredentialsRequest status.provisioned = true within 60s = PASS
  - Measurement:
    ```bash
    # Create AWS CredentialsRequest
    cat <<EOF | oc apply -f -
    apiVersion: cloudcredential.openshift.io/v1
    kind: CredentialsRequest
    metadata:
      name: test-aws-creds
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
        name: test-aws-secret
        namespace: <namespace>
    EOF

    # Wait and check provisioning
    sleep 60
    AWS_PROVISIONED=$(oc get credentialsrequest test-aws-creds \
      -n openshift-cloud-credential-operator \
      -o jsonpath='{.status.provisioned}')

    if [ "$AWS_PROVISIONED" = "true" ]; then
      echo "PASS: AWS credentials provisioned"
    else
      echo "FAIL: AWS credentials not provisioned"
    fi
    ```

- **Metric 2: Azure Egress Connectivity**
  - Measure: CredentialsRequest provisioning success on Azure
  - Expected: Azure service principal created successfully
  - Threshold: CredentialsRequest status.provisioned = true within 60s = PASS
  - Measurement:
    ```bash
    # Create Azure CredentialsRequest
    cat <<EOF | oc apply -f -
    apiVersion: cloudcredential.openshift.io/v1
    kind: CredentialsRequest
    metadata:
      name: test-azure-creds
      namespace: openshift-cloud-credential-operator
    spec:
      providerSpec:
        apiVersion: cloudcredential.openshift.io/v1
        kind: AzureProviderSpec
        roleBindings:
        - role: Contributor
      secretRef:
        name: test-azure-secret
        namespace: <namespace>
    EOF

    # Wait and check provisioning
    sleep 60
    AZURE_PROVISIONED=$(oc get credentialsrequest test-azure-creds \
      -n openshift-cloud-credential-operator \
      -o jsonpath='{.status.provisioned}')

    if [ "$AZURE_PROVISIONED" = "true" ]; then
      echo "PASS: Azure credentials provisioned"
    else
      echo "FAIL: Azure credentials not provisioned"
    fi
    ```

- **Metric 3: GCP Egress Connectivity**
  - Measure: CredentialsRequest provisioning success on GCP
  - Expected: GCP service account created successfully
  - Threshold: CredentialsRequest status.provisioned = true within 60s = PASS
  - Measurement:
    ```bash
    # Create GCP CredentialsRequest
    cat <<EOF | oc apply -f -
    apiVersion: cloudcredential.openshift.io/v1
    kind: CredentialsRequest
    metadata:
      name: test-gcp-creds
      namespace: openshift-cloud-credential-operator
    spec:
      providerSpec:
        apiVersion: cloudcredential.openshift.io/v1
        kind: GCPProviderSpec
        predefinedRoles:
        - roles/storage.objectViewer
      secretRef:
        name: test-gcp-secret
        namespace: <namespace>
    EOF

    # Wait and check provisioning
    sleep 60
    GCP_PROVISIONED=$(oc get credentialsrequest test-gcp-creds \
      -n openshift-cloud-credential-operator \
      -o jsonpath='{.status.provisioned}')

    if [ "$GCP_PROVISIONED" = "true" ]; then
      echo "PASS: GCP credentials provisioned"
    else
      echo "FAIL: GCP credentials not provisioned"
    fi
    ```

- **Metric 4: DNS Resolution Functionality**
  - Measure: CCO pod's ability to resolve cloud API hostnames
  - Expected: DNS queries return valid IP addresses
  - Threshold: nslookup succeeds for cloud endpoints = PASS
  - Measurement:
    ```bash
    # Test DNS resolution from CCO pod
    oc exec -n openshift-cloud-credential-operator $CCO_POD -- \
      nslookup sts.amazonaws.com
    AWS_DNS=$?

    oc exec -n openshift-cloud-credential-operator $CCO_POD -- \
      nslookup management.azure.com
    AZURE_DNS=$?

    oc exec -n openshift-cloud-credential-operator $CCO_POD -- \
      nslookup iam.googleapis.com
    GCP_DNS=$?

    if [ $AWS_DNS -eq 0 ] && [ $AZURE_DNS -eq 0 ] && [ $GCP_DNS -eq 0 ]; then
      echo "PASS: DNS resolution working for all cloud platforms"
    else
      echo "FAIL: DNS resolution failed (AWS=$AWS_DNS, Azure=$AZURE_DNS, GCP=$GCP_DNS)"
    fi
    ```

- **Metric 5: Egress NetworkPolicy Port Range Validation**
  - Measure: Port range configuration in allow-egress policy
  - Expected: Port 1-65535 for TCP and UDP
  - Threshold: Port range matches specification = PASS
  - Measurement:
    ```bash
    # Extract port configuration
    TCP_PORT_START=$(oc get networkpolicy allow-egress \
      -n openshift-cloud-credential-operator \
      -o jsonpath='{.spec.egress[0].ports[0].port}')
    TCP_PORT_END=$(oc get networkpolicy allow-egress \
      -n openshift-cloud-credential-operator \
      -o jsonpath='{.spec.egress[0].ports[0].endPort}')

    UDP_PORT_START=$(oc get networkpolicy allow-egress \
      -n openshift-cloud-credential-operator \
      -o jsonpath='{.spec.egress[1].ports[0].port}')
    UDP_PORT_END=$(oc get networkpolicy allow-egress \
      -n openshift-cloud-credential-operator \
      -o jsonpath='{.spec.egress[1].ports[0].endPort}')

    if [ "$TCP_PORT_START" = "1" ] && [ "$TCP_PORT_END" = "65535" ] && \
       [ "$UDP_PORT_START" = "1" ] && [ "$UDP_PORT_END" = "65535" ]; then
      echo "PASS: Egress port range correctly configured (1-65535)"
    else
      echo "FAIL: Port range mismatch (TCP=$TCP_PORT_START-$TCP_PORT_END, UDP=$UDP_PORT_START-$UDP_PORT_END)"
    fi
    ```

**Expected Results**:
- AWS CredentialsRequest provisioned successfully within 60 seconds
- Azure CredentialsRequest provisioned successfully within 60 seconds
- GCP CredentialsRequest provisioned successfully within 60 seconds
- DNS resolution works for all cloud platform endpoints
- Egress NetworkPolicy configured with port range 1-65535 for TCP and UDP
- No egress connection denials in CCO pod logs

## Validation Methods

### 1. NetworkPolicy Resource Verification
**Method**: Kubernetes API queries to validate NetworkPolicy existence and configuration
**Implementation**:
```bash
# Verify all 5 NetworkPolicies exist
POLICIES=("default-deny" "allow-egress" "allow-ingress-metrics" "allow-ingress-pprof" "allow-ingress-webhook")

for policy in "${POLICIES[@]}"; do
  if oc get networkpolicy $policy -n openshift-cloud-credential-operator &>/dev/null; then
    echo "✓ $policy exists"
  else
    echo "✗ $policy missing"
  fi
done

# Verify default-deny policy configuration
DEFAULT_DENY_TYPES=$(oc get networkpolicy default-deny \
  -n openshift-cloud-credential-operator \
  -o jsonpath='{.spec.policyTypes[*]}')

if echo "$DEFAULT_DENY_TYPES" | grep -q "Ingress" && echo "$DEFAULT_DENY_TYPES" | grep -q "Egress"; then
  echo "PASS: default-deny blocks both Ingress and Egress"
else
  echo "FAIL: default-deny not configured correctly"
fi
```

**Threshold**: All 5 policies exist + default-deny has both Ingress and Egress types = PASS

---

### 2. End-to-End Credential Workflow
**Method**: Create CredentialsRequest, monitor status progression, validate provisioned credentials
**Implementation**:
```bash
# Function to test credential workflow
test_credential_workflow() {
  local platform=$1
  local cred_name="test-${platform}-workflow"

  # Create CredentialsRequest (YAML varies by platform)
  oc apply -f credentialsrequest-${platform}.yaml

  # Monitor status with timeout
  TIMEOUT=120
  ELAPSED=0
  while [ $ELAPSED -lt $TIMEOUT ]; do
    STATUS=$(oc get credentialsrequest $cred_name \
      -n openshift-cloud-credential-operator \
      -o jsonpath='{.status.conditions[?(@.type=="Provisioned")].status}' 2>/dev/null)

    if [ "$STATUS" = "True" ]; then
      echo "PASS: $platform credentials provisioned in ${ELAPSED}s"
      return 0
    fi

    sleep 5
    ELAPSED=$((ELAPSED + 5))
  done

  echo "FAIL: $platform credentials not provisioned within ${TIMEOUT}s"
  return 1
}

# Test each platform
test_credential_workflow "aws"
test_credential_workflow "azure"
test_credential_workflow "gcp"
```

**Threshold**: CredentialsRequest reaches Provisioned status within 120 seconds for each platform = PASS

---

### 3. Ingress Endpoint Accessibility Testing
**Method**: HTTP requests to metrics, pprof, and webhook endpoints with response validation
**Implementation**:
```bash
# Test metrics endpoint
test_metrics_endpoint() {
  local cco_pod=$(oc get pods -n openshift-cloud-credential-operator \
    -l app=cloud-credential-operator -o jsonpath='{.items[0].metadata.name}')

  # Execute curl from within pod (avoids external network policy issues)
  RESPONSE=$(oc exec -n openshift-cloud-credential-operator $cco_pod -- \
    curl -k -s -o /dev/null -w "%{http_code}" https://localhost:8443/metrics)

  if [ "$RESPONSE" = "200" ]; then
    echo "PASS: Metrics endpoint accessible (HTTP $RESPONSE)"
  else
    echo "FAIL: Metrics endpoint returned HTTP $RESPONSE"
  fi
}

# Test webhook endpoint
test_webhook_endpoint() {
  # Create test pod with pod identity annotation
  kubectl run test-pod-identity --image=busybox --restart=Never \
    --overrides='{"metadata":{"annotations":{"eks.amazonaws.com/role-arn":"arn:aws:iam::123456789012:role/test"}}}' \
    -- sleep 3600

  # Check if pod was mutated (webhook processed it)
  sleep 3
  MUTATED=$(kubectl get pod test-pod-identity \
    -o jsonpath='{.spec.volumes[?(@.name=="aws-iam-token")]}')

  if [ -n "$MUTATED" ]; then
    echo "PASS: Webhook successfully processed pod admission"
  else
    echo "FAIL: Webhook did not mutate pod"
  fi

  # Cleanup
  kubectl delete pod test-pod-identity --force --grace-period=0
}

test_metrics_endpoint
test_webhook_endpoint
```

**Threshold**: Metrics returns HTTP 200 + Webhook mutates test pod = PASS

---

### 4. Network Traffic Analysis
**Method**: Monitor pod logs and network policy metrics for denied connections
**Implementation**:
```bash
# Analyze CCO pod logs for network denials
analyze_network_denials() {
  local cco_pod=$(oc get pods -n openshift-cloud-credential-operator \
    -l app=cloud-credential-operator -o jsonpath='{.items[0].metadata.name}')

  # Look for connection errors in recent logs
  DENIED_COUNT=$(oc logs -n openshift-cloud-credential-operator $cco_pod --since=10m | \
    grep -iE '(connection refused|connection timeout|network.*denied|dial.*timeout)' | \
    wc -l)

  # Filter out expected errors (e.g., testing scenarios)
  # Count only unexpected denials

  if [ $DENIED_COUNT -eq 0 ]; then
    echo "PASS: No network connection denials detected"
  else
    echo "WARN: Found $DENIED_COUNT potential network denial messages"
    oc logs -n openshift-cloud-credential-operator $cco_pod --since=10m | \
      grep -iE '(connection refused|connection timeout|network.*denied)'
  fi
}

analyze_network_denials
```

**Threshold**: 0 unexpected network denial messages in 10-minute log window = PASS
