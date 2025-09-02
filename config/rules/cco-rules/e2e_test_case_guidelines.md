# E2E Test Case Addition Guidelines for Cloud Credential Operator (CCO)

## ⚠️ CRITICAL RULE: Do Not Create New Test Files

**NEVER create new test files for CCO E2E tests.** All test cases must be added to the existing unified test file.

### File Assignment:
- **All CCO Tests** → Add to `cloudcredential.go`
- **Utility Functions** → Add to `cloudcredential_util.go`

### Why This Rule Exists:
1. **Unified Structure**: CCO uses a single file approach unlike other components
2. **Consistency**: Follows established CCO test architecture
3. **Efficiency**: Leverages existing CCO-specific setup and utilities
4. **Code Review**: Easier to review CCO-related changes in one place
5. **Execution**: Single file has proper capability checks and setup

### Correct vs Incorrect Approach:
```go
// ✅ CORRECT: Add to existing unified file
// File: cloudcredential.go
g.It("NonHyperShiftHOST-Author:username-Medium-CCO-XXXX-New CCO feature test [Serial]", func() {
    // Test implementation
})

// ❌ WRONG: Creating new file
// File: cco_new_feature.go (DO NOT CREATE)
g.It("Test description", func() {
    // Test implementation
})
```

---

## Overview
This guideline standardizes the addition and management of CCO E2E test cases, ensuring consistent structure, clear classification, and maintainable and executable tests.  
All new CCO E2E test cases must follow the rules and steps outlined below.

---

## Rule 1: Determine the Target File

CCO tests use a **unified file structure** unlike other components. All platform-specific tests are maintained in a single file:

| Test Type    | Target File      |
|------------|----------------|
| All CCO E2E tests | `cloudcredential.go` |
| Utility functions | `cloudcredential_util.go` |

### File Structure
```
cloudcredential/
├── cloudcredential.go - All CCO E2E tests (AWS, Azure, GCP, etc.)
├── cloudcredential_util.go - Utility functions and constants
```

### Standard Structure for CCO Test File
```go
package cloudcredential

import (
    // Standard imports for CCO tests
    g "github.com/onsi/ginkgo/v2"
    o "github.com/onsi/gomega"
    exutil "github.com/openshift/openshift-tests-private/test/extended/util"
    e2e "k8s.io/kubernetes/test/e2e/framework"
)

var _ = g.Describe("[sig-cco] Cluster_Operator CCO is enabled", func() {
    defer g.GinkgoRecover()
    
    var (
        oc = exutil.NewCLI("default-cco", exutil.KubeConfigPath())
        // Other standard variables
    )
    
    g.BeforeEach(func() {
        exutil.SkipNoCapabilities(oc, ccoCap)
    })
    
    // Multiple g.It() test cases
    g.It("Test case 1", func() { ... })
    g.It("Test case 2", func() { ... })
})
```

## Rule 2: Test Case Naming Convention

### Standard format for CCO tests:
```
NonHyperShiftHOST-[Platform]-[Pre/Post]-Author:[name]-[Priority]-[JIRA-KEY]-[description] [Tags]
```

### Examples from existing CCO tests:

```go
// AWS STS test example
g.It("NonHyperShiftHOST-ROSA-OSD_CCS-Author:jshu-LEVEL0-Critical-36498-CCO credentials secret change to STS-style", func() {

// Azure workload identity test example  
g.It("NonHyperShiftHOST-OSD_CCS-ARO-Author:mihuang-LEVEL0-Critical-66538-Azure workload identity cluster healthy check.", func() {

// GCP workload identity test example
g.It("Author:jshu-NonHyperShiftHOST-OSD_CCS-Critical-75429-GCP workload identity management for olm managed operators", func() {

// Platform-agnostic upgrade test
g.It("NonHyperShiftHOST-PstChkUpgrade-NonPreRelease-Author:mihuang-High-23352-Cloud credential operator resets progressing transition timestamp when it upgrades", func() {
```

### Platform Tags:
- **AWS**: `ROSA`, `OSD_CCS` 
- **Azure**: `ARO`, `OSD_CCS`
- **GCP**: `OSD_CCS`
- **Multi-platform**: `ROSA-OSD_CCS-ARO`

## Rule 3: Add a Test Case Inside the Existing Describe Block

```go
g.It("NonHyperShiftHOST-Author:yourname-Medium-[JIRA-KEY]-[description] [Serial]", func() {
    // CCO test structure follows this pattern:
    exutil.By("Step description")
    
    // Use oc.AsAdmin() for cluster-level operations
    // Use established patterns from existing tests
    
    // Common CCO operations:
    // - Check CCO operator status
    // - Validate credentials requests
    // - Test cloud provider integrations
    // - Verify STS/workload identity configurations
})
```

## Rule 4: CCO-Specific Testing Patterns

### Key Constants (from cloudcredential_util.go):
```go
const (
    ccoNs                    = "openshift-cloud-credential-operator"
    ccoCap                   = "CloudCredential"
    ccoRepo                  = "cloud-credential-operator"
    defaultSTSCloudTokenPath = "/var/run/secrets/kubernetes.io/serviceaccount/token"
    DefaultTimeout           = 120
)
```

### Common Test Patterns:
```go
// 1. Capability check (always include)
g.BeforeEach(func() {
    exutil.SkipNoCapabilities(oc, ccoCap)
})

// 2. CCO namespace operations
oc.AsAdmin().WithoutNamespace().Run("get").Args("ns", ccoNs)

// 3. Credentials request testing
oc.AsAdmin().WithoutNamespace().Run("get").Args("credentialsrequests", "-n", ccoNs)

// 4. Cloud provider credential validation
// Check for AWS STS, Azure WIF, GCP WIF tokens

// 5. CCO operator status verification
oc.AsAdmin().WithoutNamespace().Run("get").Args("co", "cloud-credential")
```

## Rule 5: Forbidden Actions & Correct Practices

### Forbidden Actions:
- Creating new test files (CCO uses unified structure in cloudcredential.go)
- Modifying cloudcredential_util.go constants without coordination
- Adding tests outside the main Describe block
- Skipping capability checks with `exutil.SkipNoCapabilities(oc, ccoCap)`

### Correct Practices:
- Add all tests to `cloudcredential.go` regardless of platform
- Use platform-specific tags in test names for identification
- Follow established utility functions from `cloudcredential_util.go`
- Include proper cleanup and error handling
- Use `oc.AsAdmin()` for cluster-level credential operations

## Rule 6: CCO Test Categories

### By Platform Integration:
- **AWS STS**: Tests for AWS Security Token Service integration
- **Azure WIF**: Tests for Azure Workload Identity Federation
- **GCP WIF**: Tests for Google Cloud Workload Identity Federation
- **Multi-cloud**: Tests that work across cloud providers

### By Functionality:
- **Credential Management**: CredentialsRequest lifecycle
- **Operator Health**: CCO operator status and upgrades
- **Security**: Pod security, read-only filesystems
- **Integration**: OLM operator credential management

## Rule 7: Test Data and Templates

### Test Data Location:
- Templates: `test/extended/testdata/cluster_operator/cloudcredential/`
- Example: `credentials_request.yaml` template for CredentialsRequest testing

### Usage Pattern:
```go
// Use template for creating test resources
testDataDir := exutil.FixturePath("testdata", "cluster_operator", "cloudcredential")
credReqTemplate := filepath.Join(testDataDir, "credentials_request.yaml")
```

## ⚠️ CRITICAL RULE: Fake Configuration for Debugging

### 🚨 MANDATORY FAKE CONFIGURATION RULE
**ALWAYS set `fake: "false"` during initial test generation and development.**

### 🔄 FAKE CONFIGURATION WORKFLOW:
1. **Initial Generation**: Set `fake: "false"` for real credential testing
2. **After Verification**: Change to `fake: "true"` for debugging and iteration
3. **Development Mode**: Use `fake: "true"` to reduce resource consumption

### 📋 FAKE CONFIGURATION IMPLEMENTATION:
```go
// CCO-specific test configuration
testConfig := ccoTestConfig{
    fake:                "false",  // ⚠️ CRITICAL: Set to "false" initially for real testing
    testName:            "CCO-XXXX-test",
    namespace:           ccoNs,
    capability:          ccoCap,
    timeout:             DefaultTimeout,
    platform:            "aws", // or "azure", "gcp"
    credentialType:      "sts", // or "wif", "manual"
    template:            filepath.Join(testDataDir, "credentials_request.yaml"),
}
```

### 🎯 FAKE CONFIGURATION BENEFITS:
- **Debugging**: Facilitate faster debugging without real credential operations
- **Resource Efficiency**: Reduce cloud resource consumption during development
- **Iteration Speed**: Enable faster test iterations and validation
- **Cost Control**: Minimize cloud costs during test development

### ⚡ FAKE CONFIGURATION SWITCHING:
```go
// For Development/Debugging: Change to "true"
fake: "true"  // Enables fake mode for faster iteration

// For Production Testing: Keep as "false"  
fake: "false" // Enables real credential operations
```

### 🔧 FAKE CONFIGURATION VALIDATION:
- **Real Testing**: `fake: "false"` → Creates actual credential operations for validation
- **Debug Mode**: `fake: "true"` → Skips credential operations for faster development
- **Always Verify**: Test with `fake: "false"` before final submission

---

## Summary

CCO E2E tests follow a **unified file approach** where all platform-specific tests coexist in `cloudcredential.go`. This differs from components like Hive that use separate platform files. The key is to use descriptive test names with platform tags and follow established CCO testing patterns for credential management across cloud providers.