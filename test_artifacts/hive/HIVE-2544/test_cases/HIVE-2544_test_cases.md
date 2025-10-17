# Test Case: HIVE-2544
**Component:** hive
**Summary:** [Hive/Azure/Azure Gov] Machinepool: Unable to generate machinesets in regions without availability zone support

## Test Overview
- **Total Test Cases:** 3
- **Test Types:** E2E, Manual
- **Estimated Time:** 60 minutes

## Test Cases

### Test Case HIVE-2544_001
**Name:** MachinePool Creation in Non-Zoned Azure Region
**Description:** Verify MachinePool creation succeeds in Azure commercial cloud regions without availability zone support
**Type:** E2E
**Priority:** High

#### Prerequisites
- Hive cluster installed and operational (version 4.12+)
- Target cluster deployed in Azure non-zoned region (e.g., centralindia, westindia, or verify region has no zones via Azure API)
- ClusterDeployment CR in Ready state with status.conditions[?(@.type=="Ready")].status=="True"
- Access to hive-controllers logs for validation
- oc CLI configured with kubeconfig for Hive cluster

#### Test Steps
1. **Action:** Identify Azure non-zoned region for target cluster
   ```bash
   # Verify cluster region
   oc get clusterdeployment <cluster-name> -n <namespace> -o jsonpath='{.spec.platform.azure.region}'

   # Verify region has no availability zones (optional - can use Azure CLI)
   # az account list-locations --query "[?name=='<region>'].availabilityZoneMappings" -o table
   ```
   **Expected:** Region confirmed as non-zoned (no availability zones listed)

2. **Action:** Create MachinePool CR without specifying zones
   ```bash
   cat <<EOF | oc apply -f -
   apiVersion: hive.openshift.io/v1
   kind: MachinePool
   metadata:
     name: test-machinepool-nonzoned
     namespace: <cluster-namespace>
   spec:
     clusterDeploymentRef:
       name: <cluster-deployment-name>
     name: infra
     platform:
       azure:
         osDisk:
           diskSizeGB: 128
         type: Standard_D4s_v3
     replicas: 3
   EOF
   ```
   **Expected:** MachinePool CR created successfully with status code 201

3. **Action:** Monitor hive-controllers machinepool controller logs for zone detection
   ```bash
   oc logs -n hive deployment/hive-controllers -c machinepool --since=2m | grep -E "(availability zones|zero zones)"
   ```
   **Expected:** Log message appears: "No availability zones detected for region. Using non-zoned deployment."
   **Expected:** No error message: "zero zones returned for region"

4. **Action:** Wait for MachinePool reconciliation and verify status
   ```bash
   # Wait for Ready condition (timeout 5 minutes)
   oc wait --for=condition=Ready machinepool/test-machinepool-nonzoned -n <namespace> --timeout=300s

   # Verify MachinePool status
   oc get machinepool test-machinepool-nonzoned -n <namespace> -o jsonpath='{.status.conditions[?(@.type=="Ready")]}'
   ```
   **Expected:** MachinePool status.conditions shows Ready=True within 5 minutes
   **Expected:** No error conditions present

5. **Action:** Verify single non-zoned machineset is created
   ```bash
   # List machinesets for this machinepool
   oc get machineset -n openshift-machine-api --selector=hive.openshift.io/machine-pool=test-machinepool-nonzoned -o json | jq '{count: .items | length, names: [.items[].metadata.name]}'

   # Verify machineset has no zone configuration
   oc get machineset -n openshift-machine-api --selector=hive.openshift.io/machine-pool=test-machinepool-nonzoned -o json | jq '.items[0].spec.template.spec.providerSpec.value.zone'
   ```
   **Expected:** Exactly 1 machineset created (count == 1)
   **Expected:** MachineSet name has no zone suffix (e.g., cluster-infra, NOT cluster-infra-1)
   **Expected:** MachineSet spec.template.spec.providerSpec.value.zone is null or empty

6. **Action:** Verify machineset replicas match MachinePool configuration
   ```bash
   oc get machineset -n openshift-machine-api --selector=hive.openshift.io/machine-pool=test-machinepool-nonzoned -o jsonpath='{.items[0].spec.replicas}'
   ```
   **Expected:** MachineSet replicas == 3 (matching MachinePool spec.replicas)

7. **Action:** Verify machines are provisioned and reach Running phase
   ```bash
   # Wait for machines to be created (timeout 10 minutes)
   sleep 30
   oc get machine -n openshift-machine-api --selector=machine.openshift.io/cluster-api-machineset=$(oc get machineset -n openshift-machine-api --selector=hive.openshift.io/machine-pool=test-machinepool-nonzoned -o jsonpath='{.items[0].metadata.name}') -o json | jq '{count: .items | length, phases: [.items[].status.phase]}'
   ```
   **Expected:** Machine count == 3 (matching machineset replicas)
   **Expected:** All machines progress to Running or Provisioned phase within 10 minutes

8. **Action:** Validate no error logs for zone-related issues
   ```bash
   oc logs -n hive deployment/hive-controllers -c machinepool --since=10m | grep -i "error.*zone" | wc -l
   ```
   **Expected:** Error count == 0 (no zone-related errors)

9. **Action:** Cleanup test resources
   ```bash
   oc delete machinepool test-machinepool-nonzoned -n <namespace>
   ```
   **Expected:** MachinePool and associated machineset deleted successfully

---

### Test Case HIVE-2544_002
**Name:** MachinePool Creation in Non-Zoned Azure Gov Region
**Description:** Verify MachinePool creation succeeds in Azure Government cloud regions without availability zone support (e.g., usgovtexas)
**Type:** Manual
**Priority:** High

#### Prerequisites
- Hive cluster configured for Azure Government cloud access
- Target cluster deployed in Azure Gov non-zoned region (usgovtexas)
- ClusterDeployment CR in Ready state
- Azure Gov credentials configured in Hive
- Access to Azure Gov portal and API endpoints for validation
- oc CLI configured with kubeconfig for Hive cluster

#### Test Steps
1. **Action:** Verify Azure Gov cluster deployment and region
   ```bash
   # Verify cluster is deployed in Azure Gov
   oc get clusterdeployment <cluster-name> -n <namespace> -o jsonpath='{.spec.platform.azure.cloudName}'

   # Verify region is usgovtexas
   oc get clusterdeployment <cluster-name> -n <namespace> -o jsonpath='{.spec.platform.azure.region}'
   ```
   **Expected:** cloudName == "AzureUSGovernmentCloud"
   **Expected:** region == "usgovtexas" (or other non-zoned Azure Gov region)

2. **Action:** Create MachinePool CR for Azure Gov cluster
   ```bash
   cat <<EOF | oc apply -f -
   apiVersion: hive.openshift.io/v1
   kind: MachinePool
   metadata:
     name: test-machinepool-azgov
     namespace: <cluster-namespace>
   spec:
     clusterDeploymentRef:
       name: <cluster-deployment-name>
     name: worker-gov
     platform:
       azure:
         osDisk:
           diskSizeGB: 128
         type: Standard_D4s_v3
     replicas: 3
   EOF
   ```
   **Expected:** MachinePool CR created successfully

3. **Action:** Monitor machinepool controller logs for Azure Gov cloud operations
   ```bash
   oc logs -n hive deployment/hive-controllers -c machinepool --since=2m | grep -E "(usgovtexas|availability zones|AzureUSGovernmentCloud)"
   ```
   **Expected:** Log message: "No availability zones detected for region. Using non-zoned deployment."
   **Expected:** Azure Gov API endpoints used (not commercial cloud endpoints)

4. **Action:** Verify MachinePool reconciliation in Azure Gov environment
   ```bash
   oc wait --for=condition=Ready machinepool/test-machinepool-azgov -n <namespace> --timeout=300s
   oc get machinepool test-machinepool-azgov -n <namespace> -o jsonpath='{.status.conditions[?(@.type=="Ready")]}'
   ```
   **Expected:** MachinePool Ready=True within 5 minutes
   **Expected:** No authentication errors with Azure Gov cloud

5. **Action:** Verify single non-zoned machineset created in Azure Gov
   ```bash
   oc get machineset -n openshift-machine-api --selector=hive.openshift.io/machine-pool=test-machinepool-azgov -o json | jq '{count: .items | length, zone: .items[0].spec.template.spec.providerSpec.value.zone}'
   ```
   **Expected:** MachineSet count == 1 (non-zoned deployment)
   **Expected:** zone field is null or empty

6. **Action:** Verify machines are provisioned in Azure Gov cloud
   ```bash
   oc get machine -n openshift-machine-api --selector=machine.openshift.io/cluster-api-machineset=$(oc get machineset -n openshift-machine-api --selector=hive.openshift.io/machine-pool=test-machinepool-azgov -o jsonpath='{.items[0].metadata.name}') -o json | jq '{count: .items | length, phases: [.items[].status.phase]}'
   ```
   **Expected:** Machine count == 3
   **Expected:** All machines reach Running/Provisioned phase

7. **Action:** Validate Azure Gov-specific configurations
   ```bash
   # Verify machine providerSpec uses Azure Gov endpoints
   oc get machine -n openshift-machine-api --selector=machine.openshift.io/cluster-api-machineset=$(oc get machineset -n openshift-machine-api --selector=hive.openshift.io/machine-pool=test-machinepool-azgov -o jsonpath='{.items[0].metadata.name}') -o json | jq '.items[0].spec.providerSpec.value' | grep -i gov
   ```
   **Expected:** Azure Gov cloud configurations present in providerSpec

8. **Action:** Cleanup test resources
   ```bash
   oc delete machinepool test-machinepool-azgov -n <namespace>
   ```
   **Expected:** Resources deleted successfully

---

### Test Case HIVE-2544_003
**Name:** MachinePool in Zoned Azure Region - Regression Test
**Description:** Verify existing zone-based MachinePool functionality remains intact in Azure regions with availability zone support
**Type:** E2E
**Priority:** High

#### Prerequisites
- Hive cluster installed and operational (version 4.12+)
- Target cluster deployed in Azure zoned region (e.g., eastus, westeurope, or other multi-zone region)
- ClusterDeployment CR in Ready state
- Region supports availability zones (verify via Azure API or documentation)
- oc CLI configured with kubeconfig for Hive cluster

#### Test Steps
1. **Action:** Verify cluster is deployed in Azure zoned region
   ```bash
   # Get cluster region
   oc get clusterdeployment <cluster-name> -n <namespace> -o jsonpath='{.spec.platform.azure.region}'

   # Optionally verify region supports zones using Azure CLI
   # az account list-locations --query "[?name=='<region>'].availabilityZoneMappings" -o table
   ```
   **Expected:** Region confirmed as multi-zone (typically 3 availability zones)

2. **Action:** Create MachinePool CR without specifying zones (auto-detection)
   ```bash
   cat <<EOF | oc apply -f -
   apiVersion: hive.openshift.io/v1
   kind: MachinePool
   metadata:
     name: test-machinepool-zoned
     namespace: <cluster-namespace>
   spec:
     clusterDeploymentRef:
       name: <cluster-deployment-name>
     name: worker-zoned
     platform:
       azure:
         osDisk:
           diskSizeGB: 128
         type: Standard_D4s_v3
     replicas: 6
   EOF
   ```
   **Expected:** MachinePool CR created successfully

3. **Action:** Wait for MachinePool reconciliation
   ```bash
   oc wait --for=condition=Ready machinepool/test-machinepool-zoned -n <namespace> --timeout=300s
   oc get machinepool test-machinepool-zoned -n <namespace> -o jsonpath='{.status.conditions[?(@.type=="Ready")]}'
   ```
   **Expected:** MachinePool Ready=True within 5 minutes

4. **Action:** Verify multiple zone-specific machinesets are created
   ```bash
   # Count machinesets and list their names
   oc get machineset -n openshift-machine-api --selector=hive.openshift.io/machine-pool=test-machinepool-zoned -o json | jq '{count: .items | length, names: [.items[].metadata.name]}'

   # Verify each machineset has zone assignment
   oc get machineset -n openshift-machine-api --selector=hive.openshift.io/machine-pool=test-machinepool-zoned -o json | jq '[.items[] | {name: .metadata.name, zone: .spec.template.spec.providerSpec.value.zone}]'
   ```
   **Expected:** MachineSet count >= 2 (typically 3 for most Azure regions)
   **Expected:** Each machineset has zone-specific suffix (e.g., cluster-worker-zoned-1, cluster-worker-zoned-2, cluster-worker-zoned-3)
   **Expected:** Each machineset has unique zone assignment in spec.template.spec.providerSpec.value.zone

5. **Action:** Verify total replicas are distributed across zones
   ```bash
   # Get replica distribution
   oc get machineset -n openshift-machine-api --selector=hive.openshift.io/machine-pool=test-machinepool-zoned -o json | jq '[.items[] | {name: .metadata.name, replicas: .spec.replicas}]'

   # Calculate total replicas
   oc get machineset -n openshift-machine-api --selector=hive.openshift.io/machine-pool=test-machinepool-zoned -o json | jq '[.items[].spec.replicas] | add'
   ```
   **Expected:** Sum of all machineset replicas == 6 (matching MachinePool spec.replicas)
   **Expected:** Replicas distributed evenly across zones (e.g., 2 per zone for 3 zones)

6. **Action:** Verify machines are created across all zones
   ```bash
   # Get machines grouped by machineset
   oc get machine -n openshift-machine-api --selector=hive.openshift.io/machine-pool=test-machinepool-zoned -o json | jq '{total_count: .items | length, by_zone: [group_by(.spec.providerSpec.value.zone)[] | {zone: .[0].spec.providerSpec.value.zone, count: length}]}'
   ```
   **Expected:** Total machine count == 6
   **Expected:** Machines distributed across multiple zones
   **Expected:** All machines reach Running/Provisioned phase

7. **Action:** Verify no regression in zone-based deployment logging
   ```bash
   oc logs -n hive deployment/hive-controllers -c machinepool --since=10m | grep -E "(zone|test-machinepool-zoned)" | grep -v "zero zones"
   ```
   **Expected:** No "zero zones returned" error messages
   **Expected:** No errors related to zone detection or assignment

8. **Action:** Cleanup test resources
   ```bash
   oc delete machinepool test-machinepool-zoned -n <namespace>
   ```
   **Expected:** MachinePool and all zone-specific machinesets deleted successfully
