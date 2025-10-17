# Test Requirements Analysis

## Component Name
hive

## Card Summary
**JIRA**: HIVE-2544
**Title**: [Hive/Azure/Azure Gov] Machinepool: Unable to generate machinesets in regions without availability zone support

**Problem**: When creating a MachinePool using Hive in Azure/Azure Gov regions that don't support availability zones, the machinepool controller fails to generate machinesets with error "zero zones returned for region".

**Solution**: Modified azureactuator.go to handle regions without availability zones gracefully:
- If no zones are specified and the region has no zones, pass through "no zones" to allow non-zoned deployment
- Log informational message for visibility when deploying in non-zoned regions
- Remove error condition that blocked machinepool creation in regions without AZ support

**Affected Versions**: 4.12.z, 4.13.z, 4.14.z, 4.15.z, 4.16, 4.17

## Test Requirements

### Functional Requirements
1. **Non-zoned Region Support**: MachinePool creation must succeed in Azure regions without availability zone support
2. **Zone Handling Logic**: Controller must detect when region has zero zones and proceed with non-zoned deployment
3. **Logging**: System must log informational message when deploying in non-zoned regions
4. **Backward Compatibility**: Existing zone-based deployments must continue to work normally

### Technical Requirements
1. Verify azureactuator.go correctly handles empty zones array
2. Verify non-zoned deployment creates single machineset (not zone-specific machinesets)
3. Verify logging message appears: "No availability zones detected for region. Using non-zoned deployment."
4. Verify no error returned when len(zones) == 0

### Integration Requirements
1. MachineSet generation must work with OpenShift installer's Azure platform logic
2. Non-zoned machinepool must integrate correctly with existing cluster infrastructure
3. Both Azure commercial and Azure Gov clouds must be supported

## Affected Platforms
- Azure (commercial cloud)
- Azure Gov (government cloud)
- Regions without availability zone support (e.g., usgovtexas)

## Test Scenarios

### Scenario 1: MachinePool in Non-Zoned Azure Region
**Given**: Hive cluster deployed in Azure region without AZ support (e.g., usgovtexas)
**When**: User creates a MachinePool without specifying zones
**Then**:
- MachinePool creation succeeds
- Single non-zoned machineset is generated
- Log message appears: "No availability zones detected for region. Using non-zoned deployment."
- No error about "zero zones returned"

### Scenario 2: MachinePool in Zoned Azure Region (Regression Test)
**Given**: Hive cluster deployed in Azure region with AZ support (e.g., eastus)
**When**: User creates a MachinePool without specifying zones
**Then**:
- MachinePool creation succeeds
- Multiple zone-specific machinesets are generated (one per zone)
- Zones are auto-detected and assigned

### Scenario 3: MachinePool with Explicit Zones in Non-Zoned Region
**Given**: Hive cluster deployed in Azure region without AZ support
**When**: User creates a MachinePool and specifies zones explicitly
**Then**:
- System behavior depends on implementation (graceful degradation vs strict error)
- Current implementation: zones are ignored, non-zoned deployment proceeds with warning log

### Scenario 4: Azure Gov Cloud Non-Zoned Region
**Given**: Hive cluster deployed in Azure Gov cloud in non-zoned region (usgovtexas)
**When**: User creates a MachinePool
**Then**:
- MachinePool creation succeeds
- Non-zoned machineset is generated
- Azure Gov-specific configurations are applied correctly

## Edge Cases

1. **Region Migration**: If Azure adds AZ support to previously non-zoned region, existing machinepools should remain stable
2. **Mixed Deployments**: Cluster with both zoned and non-zoned machinepools (different regions)
3. **Zone Detection Failure**: If Azure API fails to return zone information, system should handle gracefully
4. **Empty Zones Array vs Nil Zones**: Code must handle both empty array [] and nil zones field
5. **Cross-Region Consistency**: Verify behavior consistency between Azure commercial and Azure Gov clouds
