# 🔄 Test Workflow Comprehensive Guide

## 📋 Overview

This guide covers two complete workflows:

1. **Test Case Generation Workflow** - Generate test cases and E2E test code from JIRA tickets
2. **Test Execution Workflow** - Execute generated test cases and analyze results

## 🚀 Test Case Generation Workflow

### Workflow Description
This is the **Test Case Generation Workflow** - dedicated to generating comprehensive test cases and E2E test code from JIRA tickets. This workflow does NOT execute tests; it only generates test artifacts.

### Execute Test Case Generation Workflow

**Input**: JIRA issue key (e.g., "HIVE-2883")

**Configuration**: Load from `config/test_case_generation_workflow.yaml`

**Workflow Phases**:

#### Phase 1: Requirements Gathering Agent
**Execute**: requirements_gathering agent
**Output**: `test_requirements_output.yaml`

#### Phase 2: Test Strategy Analysis Agent
**Execute**: test_strategy_analysis agent  
**Input**: Phase 1 output
**Output**: `{JIRA_KEY}_test_strategy_output.yaml`

#### Phase 3: Test Case Generation Agent
**Execute**: test_case_generation agent (Checkpoint Aggregator + XML/Markdown generation)
**Input**: Phase 1 & 2 outputs
**Output**: 
- `{JIRA_KEY}_polarion.xml`
- `{JIRA_KEY}_test_case.md`

#### Phase 4: E2E Test Generation Agent
**Execute**: e2e_test_generation agent
**Input**: Phase 2 & 3 outputs
**Output**: `{JIRA_KEY}_e2e_test_results.md` + actual test files in repository

**Execution Order**: Phase 1 → Phase 2 → Phase 3 → Phase 4 (Sequential - E2E based on Phase 3 results)

**Config**: Load from config/test_case_generation_workflow.yaml
**Rules**: Load from config/rules/{COMPONENT}-rules/
**Templates**: Load from config/templates/
**Checklist**: Follow prompts/check_list.md for session setup
**Template Compliance**: ALWAYS follow template structure exactly - no deviations allowed

**Time**: 4 minutes total (optimized for generation only)

## 🧪 Test Execution Workflow

### Workflow Description
This is the **Test Execution Workflow** - dedicated to executing tests using previously generated test cases. This workflow requires test artifacts from the Test Case Generation Workflow to be completed first.

### Execute Test Execution Workflow

**Prerequisites**: Test Case Generation Workflow must be completed successfully

**Input**: JIRA issue key (e.g., "HIVE-2883") + generated test artifacts

**Configuration**: Load from `config/test_execution_workflow.yaml`

**Workflow Phases**:

#### Phase 1: Environment Verification Agent
**Execute**: environment_checker agent
**Purpose**: Verify test environment and prerequisites
**Input**: Generated test case artifacts
**Output**: `environment_check_results.yaml`
**Timeout**: 60s

#### Phase 2: E2E Test Execution Agent
**Execute**: e2e_test_runner agent
**Purpose**: Execute E2E tests in target environment
**Input**: Phase 1 output + E2E test files from generation workflow
**Output**: 
- `e2e_execution_log.txt`
- `e2e_test_results.xml`
**Timeout**: 480s (8 minutes)

#### Phase 3: Manual Test Guidance Agent (Parallel)
**Execute**: test_case_generation agent (guidance mode)
**Purpose**: Provide manual test execution guidance
**Input**: Manual test cases from generation workflow
**Output**: `manual_test_guidance.md`
**Timeout**: 120s
**Runs parallel with**: Phase 2

#### Phase 4: Test Result Analysis Agent
**Execute**: test_result_analyzer agent
**Purpose**: Analyze execution results and generate reports
**Input**: Phase 2 & 3 outputs
**Output**:
- `{JIRA_KEY}_execution_report.md`
- `{JIRA_KEY}_test_summary.yaml`
**Timeout**: 180s

**Execution Order**: Phase 1 → (Phase 2 || Phase 3) → Phase 4

**Config**: Load from config/test_execution_workflow.yaml
**Environment**: Requires active cluster connection and component deployment
**Template Compliance**: Follow execution result templates

**Time**: 15 minutes total (includes actual test execution)

## ⚠️ CRITICAL RULES - NEVER OMIT

### ❌ NEVER DO:
1. Skip any phase - ALL phases MUST be executed
2. Use hardcoded paths - ALWAYS use dynamic variables {COMPONENT}, {JIRA_KEY}
3. Ignore component-specific rules - ALWAYS load from config/rules/{COMPONENT}-rules/
4. Generate tests without analyzing existing patterns
5. Create files without proper directory structure
6. Skip error handling and validation

### ✅ ALWAYS DO:
1. **Load component configuration FIRST** before any operation
2. **Validate all input files exist** before processing
3. **Use dynamic variable replacement** for all paths
4. **Follow component-specific rules** exactly as defined
5. **Generate structured output** for all phases (YAML for phases 1-2, XML/Markdown for phase 3, Markdown for phase 4)
6. **Create proper directory structure** before file creation
7. **Validate output files** after generation
8. **Handle errors gracefully** with fallback strategies

## 🔍 MANDATORY VALIDATION STEPS

### Test Case Generation Workflow:
1. **Pre-execution**: Verify component exists in config/components.yaml
2. **Phase 1**: Validate JIRA issue exists and is accessible
3. **Phase 2**: Verify Phase 1 output file exists before processing
4. **Phase 3**: Verify Phase 2 strategy output exists
5. **Phase 4**: Verify Phase 3 test cases exist and E2E repository is accessible

### Test Execution Workflow:
1. **Pre-execution**: Verify all test case generation outputs exist
2. **Phase 1**: Validate cluster connectivity and component status
3. **Phase 2**: Monitor test execution and collect real-time logs
4. **Phase 3**: Ensure manual test guidance is comprehensive
5. **Phase 4**: Validate all execution artifacts are collected

## 🛡️ MANDATORY ERROR HANDLING

### Test Case Generation Workflow:
1. **Missing component**: Use default component (hive) and log warning
2. **Missing rules**: Use fallback rules from config/rules/hive-rules/
3. **Missing templates**: Use base templates from config/templates/
4. **File creation failure**: Retry up to 3 times with exponential backoff
5. **Timeout**: Fail fast if any phase exceeds its timeout limit

### Test Execution Workflow:
1. **Environment unavailable**: Fail fast and provide clear guidance
2. **Test failures**: Continue execution but mark failures for analysis
3. **Resource constraints**: Scale down test parallelism
4. **Timeout exceeded**: Gracefully terminate and preserve partial results
5. **Cleanup failure**: Log warnings but don't fail the workflow

## 📤 MANDATORY OUTPUT REQUIREMENTS

### Test Case Generation Workflow:
1. **All YAML files MUST be valid** and parseable
2. **All paths MUST use dynamic variables** {COMPONENT}, {JIRA_KEY}
3. **All output files MUST be created** in correct locations
4. **All directory structures MUST be created** before file creation
5. **All generated code MUST follow** component-specific patterns

### Test Execution Workflow:
1. **All execution logs MUST be preserved** for debugging
2. **All test results MUST be in standard format** (JUnit XML)
3. **All failure diagnostics MUST be collected** (screenshots, dumps)
4. **All execution artifacts MUST be timestamped** and versioned
5. **All reports MUST include** pass/fail counts and failure analysis

## ⚡ MANDATORY PERFORMANCE REQUIREMENTS

### Test Case Generation Workflow:
1. **Total execution time MUST NOT exceed** 5 minutes
2. **Each phase MUST complete** within its allocated time
3. **Parallel operations MUST be used** where possible
4. **Resource usage MUST be optimized** for efficiency

### Test Execution Workflow:
1. **Individual test timeout MUST NOT exceed** 5 minutes
2. **E2E tests MUST complete** within 8-10 minutes
3. **Manual tests MUST complete** within 3-5 minutes
4. **Result analysis MUST complete** within 1-3 minutes

## 🔒 MANDATORY SECURITY REQUIREMENTS

### General Requirements:
1. **Never expose sensitive data** in output files
2. **Validate all input paths** to prevent path traversal
3. **Use safe file operations** with proper permissions
4. **Log operations securely** without sensitive information

### Test Execution Specific Requirements:
1. **Clean up test resources** after test execution
2. **Execute tests in non-production environments** unless approved
3. **Isolate test execution** to prevent interference

## 🎯 Success Criteria

### Test Case Generation Workflow Success:
- ✅ All 4 phases completed successfully
- ✅ All necessary output files generated
- ✅ File format and structure correct
- ✅ Component-specific rules followed
- ✅ Completed within time limits

### Test Execution Workflow Success:
- ✅ Environment verification passed
- ✅ Tests executed successfully
- ✅ All execution artifacts collected
- ✅ Comprehensive execution report generated
- ✅ Completed within time limits

## 🔗 Workflow Integration

### Workflow Dependencies:
```
Test Case Generation Workflow → Test Execution Workflow
     ↓                           ↓
    Output Artifacts           Execution Results
```

### Configuration Dependencies:
- Both workflows depend on `config/components.yaml`
- Test execution workflow depends on test case generation workflow outputs
- Both workflows use component-specific rules and templates

### Execution Sequence:
1. Complete test case generation workflow first
2. Verify all output artifacts exist
3. Then execute test execution workflow
4. Analyze final results and reports
