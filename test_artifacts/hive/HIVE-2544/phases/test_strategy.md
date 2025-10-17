# Test Strategy

## Test Coverage Matrix

### Scenario 1: Non-Zoned Region MachinePool Creation
| Test Focus | Validation Method |
|------------|-------------------|
| **MachinePool Creation** | MachinePool CR status shows Ready=True |
| **MachineSet Generation** | Single non-zoned machineset created (no zone suffix) |
| **Logging Validation** | Log contains: "No availability zones detected for region. Using non-zoned deployment." |
| **Error Prevention** | No "zero zones returned for region" error in logs |
| **Machine Provisioning** | Machines created from machineset reach Running phase |

### Scenario 2: Zoned Region MachinePool Creation (Regression)
| Test Focus | Validation Method |
|------------|-------------------|
| **Zone Detection** | Azure zones auto-detected for region |
| **MachineSet Generation** | Multiple zone-specific machinesets created (one per zone) |
| **Zone Distribution** | Machines distributed across availability zones |
| **Existing Functionality** | No regression in zone-based deployments |

### Scenario 3: Azure Gov Cloud Non-Zoned Region
| Test Focus | Validation Method |
|------------|-------------------|
| **Cloud Compatibility** | MachinePool creation succeeds in Azure Gov |
| **Non-Zoned Deployment** | Single machineset generated in usgovtexas region |
| **Logging Validation** | Informational log message appears |
| **Gov-Specific Config** | Azure Gov authentication and API endpoints work correctly |

## Test Scenarios

### Scenario 1: MachinePool in Non-Zoned Azure Region
**Objective**: Verify MachinePool creation succeeds in Azure regions without availability zone support

**Preconditions**:
- Hive cluster installed and operational
- Target cluster deployed in Azure non-zoned region (e.g., usgovtexas)
- ClusterDeployment CR in Ready state

**Test Steps**:
1. Create MachinePool CR without specifying zones
2. Monitor machinepool controller logs for zone detection
3. Wait for MachinePool reconciliation (timeout: 5 minutes)
4. Verify single non-zoned machineset is created
5. Verify machines are provisioned from machineset
6. Check machinepool controller logs for informational message

**Expected Results**:
- MachinePool CR status.conditions shows Ready=True
- One machineset created without zone suffix (e.g., cluster-infra, not cluster-infra-1)
- Log message: "No availability zones detected for region. Using non-zoned deployment."
- No error: "zero zones returned for region"
- Machineset spec.replicas matches MachinePool spec.replicas
- Machines reach Running phase within timeout

**Quantitative Validation**:
- MachineSet count == 1 (non-zoned deployment)
- Machine count == MachinePool.spec.replicas
- Log pattern match count == 1 for informational message
- Error pattern match count == 0 for "zero zones" error

### Scenario 2: MachinePool in Zoned Azure Region (Regression Test)
**Objective**: Verify existing zone-based MachinePool functionality remains intact

**Preconditions**:
- Hive cluster installed and operational
- Target cluster deployed in Azure zoned region (e.g., eastus)
- Region supports availability zones (typically 3 zones)

**Test Steps**:
1. Create MachinePool CR without specifying zones
2. Wait for zone auto-detection
3. Monitor MachineSet creation
4. Verify zone distribution across machinesets

**Expected Results**:
- Multiple machinesets created (one per availability zone)
- Each machineset has zone-specific suffix (e.g., cluster-infra-1, cluster-infra-2, cluster-infra-3)
- Machines distributed across zones
- Total machines across all zones == MachinePool.spec.replicas

**Quantitative Validation**:
- MachineSet count == number of zones in region (typically 3)
- Sum of all machineset replicas == MachinePool.spec.replicas
- Each machineset has unique zone assignment

### Scenario 3: Azure Gov Cloud Non-Zoned Region
**Objective**: Verify MachinePool creation in Azure Government cloud non-zoned regions

**Preconditions**:
- Hive cluster configured for Azure Gov cloud
- Target cluster deployed in Azure Gov non-zoned region (usgovtexas)
- Azure Gov credentials configured correctly

**Test Steps**:
1. Create MachinePool CR for Azure Gov cluster
2. Monitor machinepool controller logs
3. Verify non-zoned deployment behavior
4. Validate Azure Gov-specific configurations

**Expected Results**:
- MachinePool creation succeeds in Azure Gov environment
- Single non-zoned machineset created
- Azure Gov API endpoints used correctly
- Gov-specific authentication handled properly

**Quantitative Validation**:
- MachineSet count == 1
- Machine count == MachinePool.spec.replicas
- No authentication errors in logs

## Validation Methods

### 1. Resource Status Validation
**Method**: Query Kubernetes resource status and conditions
```bash
oc get machinepool <name> -n <namespace> -o jsonpath='{.status.conditions[?(@.type=="Ready")].status}'
oc get machineset -n openshift-machine-api -o json | jq '.items | length'
oc get machine -n openshift-machine-api --selector=machine.openshift.io/cluster-api-machineset=<machineset-name> -o json | jq '.items | length'
```

### 2. Log Analysis
**Method**: Search controller logs for specific patterns
```bash
oc logs -n hive deployment/hive-controllers -c machinepool --since=5m | grep "No availability zones detected"
oc logs -n hive deployment/hive-controllers -c machinepool --since=5m | grep "zero zones returned" | wc -l
```

### 3. MachineSet Inspection
**Method**: Verify machineset naming and zone configuration
```bash
oc get machineset -n openshift-machine-api -o json | jq '.items[].metadata.name'
oc get machineset -n openshift-machine-api -o json | jq '.items[].spec.template.spec.providerSpec.value.zone'
```

### 4. Quantitative Metrics
**Metrics**:
- MachineSet count (1 for non-zoned, 3+ for zoned)
- Machine count per machineset
- Total machine count vs MachinePool replicas
- Log pattern match counts (informational message vs error)
- Status condition transitions (Progressing → Ready)

### 5. Time-Based Validation
**Method**: Monitor resource reconciliation within time limits
- MachinePool reconciliation timeout: 5 minutes
- Machine provisioning timeout: 10 minutes
- Status condition update interval: 30 seconds
