# 🚀 Test Case Generation Execution Guide

## 📊 Execution Mode Overview

This guide provides two execution modes to meet different performance requirements:

- **Standard Mode**: Complete in 4 minutes (240 seconds) - Suitable for production environments
- **Fast Mode**: Complete in 90 seconds (1.5 minutes) - Suitable for development and testing

## 🎯 Standard Execution Mode (4 minutes)

### Overall Time Allocation

**Total Goal: 240 seconds to complete all 4 phases**

```
Phase Timeline:
Phase 1: 0-60s    (Requirements Gathering)
Phase 2: 60-120s  (Test Strategy Analysis) 
Phase 3 & 4: 120-240s (Parallel execution of Test Case Generation + E2E Test Generation)
```

### Parallel Execution Strategy

#### Phase 1 (0-60 seconds)
- **Task**: JIRA data retrieval + requirements extraction
- **Tools**: `mcp__jira-mcp-snowflake__get_jira_issue_details`
- **Output**: `test_requirements_output.yaml`
- **Timeout**: 60 seconds

#### Phase 2 (60-120 seconds)
- **Task**: Load Phase 1 output + strategy analysis
- **Dependencies**: Wait for Phase 1 completion
- **Output**: `{JIRA_KEY}_test_strategy_output.yaml`
- **Timeout**: 60 seconds

#### Phase 3 & 4 (120-240 seconds) - **Critical: Must execute in parallel**
**Phase 3 (Test Case Generation):**
- **Task**: Generate Polarion XML + Markdown test cases
- **Tools**: DeepWiki queries + template processing
- **Output**: `_polarion.xml`, `_test_case.md`

**Phase 4 (E2E Test Generation):**
- **Task**: Generate E2E test documentation
- **Tools**: E2E pattern analysis + code pattern generation
- **Output**: `_e2e_test_results.md`

### Methods to Implement Parallel Execution

#### Method 1: Single Message with Multiple Tool Calls
```
After Phase 2 completion, immediately send a single message containing:
1. Write Phase 3 output files (parallel processing)
2. Write Phase 4 output files (parallel processing)
```

#### Method 2: Batch File Operations
```
Use MultiEdit or batch Write operations to create simultaneously:
- CCO-681_polarion.xml
- CCO-681_test_case.md  
- CCO-681_e2e_test_results.md
```

## ⚡ Fast Execution Mode (90 seconds)

### Ultra-Fast Performance Metrics
- **Total Execution Time**: 90 seconds (1.5 minutes)
- **Phase 1**: 20 seconds (Ultra-Fast Requirements Gathering)
- **Phase 2**: 15 seconds (Ultra-Fast Strategy Analysis)
- **Phase 3 & 4**: 25 seconds (Parallel Test Case + E2E Generation)
- **Buffer Time**: 30 seconds
- **Speed Improvement**: 50% faster than standard mode

### Key Optimization Points

#### 1. Simplified Component Detection
- Use fast component mapping
- Direct HIVE/CCO/OCPBUGS prefix detection
- Reduce complex content analysis

#### 2. Streamlined Phase 4
- Generate E2E documentation guidance only
- Remove actual code generation
- Single DeepWiki query limit
- 15-second timeout limit

#### 3. Performance Configuration Optimization
- Memory limit: 2GB (from 4GB)
- CPU limit: 4 cores (from 8 cores)
- Disable retry mechanism
- Enable parallel file operations

## ✅ Execution Checklist

### Standard Mode Completion Markers

#### ✅ Phase 1 Completion Markers
- [ ] JIRA issue data retrieved successfully
- [ ] test_requirements_output.yaml file generated
- [ ] File size > 30 lines
- [ ] Execution time ≤ 60 seconds

#### ✅ Phase 2 Completion Markers  
- [ ] Phase 1 output loaded successfully
- [ ] Component rules applied successfully
- [ ] test_strategy_output.yaml generated
- [ ] E2E decision clear (YES/NO)
- [ ] Execution time ≤ 60 seconds

#### ✅ Phase 3&4 Parallel Execution Markers
- [ ] **Simultaneously start** both phase processing
- [ ] Phase 3: Generate XML + Markdown files
- [ ] Phase 4: Generate E2E documentation
- [ ] Both phases complete within 120 seconds
- [ ] No inter-phase waiting time

### Fast Mode Completion Markers

#### ✅ Phase 1 (20 seconds)
- [x] Parallel JIRA data retrieval + rule loading
- [x] Cached component rules
- [x] Ultra-fast template processing
- [x] Parallel processing enabled

#### ✅ Phase 2 (15 seconds)
- [x] Rule-based binary decisions
- [x] No complex analysis
- [x] Pre-loaded templates
- [x] Ultra-fast strategy selection

#### ✅ Phase 3 & 4 (25 seconds - Parallel)
- [x] TRUE PARALLEL execution
- [x] Concurrent generation of 3 test files
- [x] Parallel E2E documentation
- [x] No inter-dependencies
- [x] Maximum CPU utilization

## 🛠️ Using Optimized Configuration

```bash
# Use ultra-fast configuration
config/test_case_generation_workflow.yaml (v3.0)
config/agents/requirements_gathering_ultra_fast.yaml
config/agents/test_strategy_analysis_ultra_fast.yaml
config/agents/test_case_generation_ultra_fast.yaml

# Ultra-fast mode enabled
performance.ultra_fast_mode.enabled: true
performance.ultra_fast_mode.parallel_phases: ["phase_3", "phase_4"]
performance.ultra_fast_mode.concurrent_file_generation: true
performance.optimization.parallel_agent_execution: true
performance.optimization.aggressive_caching: true
performance.optimization.skip_validation: true
```

## 📝 Quality Assurance

### Standard Mode
- All necessary output files generated
- Template structure strictly followed
- Core functionality complete
- Complete validation and error handling

### Fast Mode
- All necessary output files still generated
- Template structure strictly followed
- Core functionality maintained
- Only time-intensive operations removed

## 🚨 Important Notes

### Standard Mode
- Complete E2E code generation
- Retry mechanism enabled
- Requires stable network connection
- JIRA and DeepWiki queries need reliability

### Fast Mode
- Phase 4 no longer generates actual E2E code
- Retry mechanism disabled for speed
- Requires stable network connection
- JIRA and DeepWiki queries need reliability

## 📊 Expected Performance Metrics

### Standard Mode
```
Ideal execution time:
Phase 1: 45-60 seconds
Phase 2: 30-45 seconds  
Phase 3 & 4: 90-120 seconds
Total time: 240 seconds (4 minutes)
```

### Fast Mode
```
Ideal execution time:
Phase 1: 15-20 seconds
Phase 2: 10-15 seconds  
Phase 3 & 4: 20-25 seconds
Total time: 90 seconds (1.5 minutes)
```

## 🔧 Performance Optimization Measures

### Caching Strategy
- Component rules cache: 5-minute TTL
- DeepWiki query cache: 10-minute TTL  
- Template preloading: Load all templates at startup

### Parallel Optimization
- File I/O parallel: Write multiple output files simultaneously
- Network calls parallel: Concurrent DeepWiki queries
- Template processing parallel: Generate XML and Markdown simultaneously

### Error Handling
- Fast fail: Stop immediately on timeout
- No retry: Avoid time waste
- Fault tolerance mode: Continue other tasks on partial failure
