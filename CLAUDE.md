# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the **Test Case Generation Workflow** - an AI-powered system for generating comprehensive test cases for OpenShift components, specifically Hive and Cloud Credential Operator (CCO). The system uses a 4-phase workflow engine to automatically generate test documentation and E2E test code from JIRA tickets.

## Core Architecture

### Multi-Component Framework
- **Components Supported**: OpenShift Hive (`HIVE-*` tickets), Cloud Credential Operator (`CCO-*` tickets)
- **Configuration-Driven**: All behavior controlled through YAML configurations in `config/`
- **Agent-Based**: 10 specialized agents handle different aspects of test generation and execution
- **Template-Driven**: Standardized output formats with component-specific variations

### 4-Phase Workflow Engine
1. **Requirements Gathering** - Extract test requirements from JIRA tickets
2. **Test Strategy Analysis** - Determine optimal test execution strategy  
3. **Test Case Generation** - Aggregate checkpoints and generate executable test scenarios
4. **E2E Test Generation** - Generate and integrate E2E test code into openshift-tests-private repository

**Execution Order**: Phase 1 → Phase 2 → Phase 3 → Phase 4 (Sequential with some parallel execution possible)

### Configuration System
- **Workflow Configuration**: `config/test_case_generation_workflow.yaml` - Controls execution flow, phases, timeouts, and dependencies
- **Component Configuration**: `config/components.yaml` - Defines supported components, detection rules, repository paths, and mappings
- **Agent Rules**: `config/rules/{COMPONENT}-rules/component_rules_config.yaml` - Component-specific agent rule mapping
- **Agent Configurations**: `config/agents/` - Individual agent configurations with roles, tasks, and tools
- **Templates**: `config/templates/` - Standardized output templates for all phases

## Essential Commands

### Test Case Generation Workflow
```bash
# Execute complete 4-phase workflow for a JIRA ticket
# Input: JIRA issue key (e.g., "HIVE-2883", "CCO-1234")
# Follow prompts/workflow_guide.md for execution steps

# MANDATORY: All 4 phases executed with parallel optimization
# Total execution time: 90 seconds (1.5 minutes) ultra-fast with parallel processing
# 1. Phase 1 (20s): Ultra-fast requirements gathering from JIRA
# 2. Phase 2 (15s): Ultra-fast strategy analysis (depends on Phase 1)
# 3. Phase 3 & 4 (25s): Parallel test case + E2E generation (depends on Phase 2)
# Speed improvement: 50% faster than previous version
```

### Configuration Management
```bash
# Validate component configuration
cat config/components.yaml

# Check workflow engine settings
cat config/test_case_generation_workflow.yaml

# View component-specific rules
ls config/rules/hive-rules/
ls config/rules/cco-rules/
ls config/rules/common/

# Examine output templates
ls config/templates/

# Check agent configurations
ls config/agents/

# View workflow prompts
ls prompts/

# Check fast execution guide
cat prompts/execution_guide.md
```

### Agent Configuration
```bash
# View all agent configurations
ls config/agents/

# Core workflow agents
cat config/agents/requirements_gathering.yaml
cat config/agents/test_strategy_analysis.yaml
cat config/agents/test_case_generation.yaml
cat config/agents/e2e_test_generation.yaml

# Additional specialized agents
cat config/agents/e2e_code_generator.yaml
cat config/agents/e2e_validator_integrator.yaml
cat config/agents/environment_checker.yaml
cat config/agents/test_result_analyzer.yaml
cat config/agents/e2e_test_runner.yaml
cat config/agents/test-executor.yaml
```

### Workflow Validation
```bash
# Check workflow checklist
cat prompts/check_list.md

# View execution workflow prompt
cat prompts/workflow_guide.md

# Check test execution workflow
cat prompts/test_execution_workflow_prompt.md
```

## Component-Specific Architecture

### OpenShift Hive Testing
- **JIRA Prefix**: `HIVE-*`
- **Focus Areas**: Cluster provisioning, deletion, updates, hibernation
- **E2E Repository**: `/Users/mihuang/auto-test-case/openshift-tests-private` (configurable)
- **Component Repository**: `github.com/openshift/hive`
- **Rules**: `config/rules/hive-rules/` - Operator testing, day1/day2 operations
- **Test Patterns**: ClusterDeployment management, MachinePool operations
- **Examples**: `config/examples/hive-examples/` - Output format examples

### Cloud Credential Operator Testing  
- **JIRA Prefix**: `CCO-*`
- **Focus Areas**: Credential request/provisioning/rotation, multi-cloud support
- **E2E Repository**: `/Users/mihuang/auto-test-case/openshift-tests-private` (shared with Hive)
- **Component Repository**: `github.com/openshift/cloud-credential-operator`
- **Rules**: `config/rules/cco-rules/` - Credential management testing
- **Test Patterns**: CredentialsRequest workflows, cloud provider integration
- **Examples**: `config/examples/cco-examples/` - Output format examples

## Workflow Execution Requirements

### Mandatory Pre-Execution Checks
1. Load component configuration from `config/components.yaml`
2. Validate JIRA issue exists and is accessible using `jira-mcp-snowflake`
3. Verify e2e repository paths are accessible and writable
4. Initialize dynamic variables: `{COMPONENT}`, `{JIRA_KEY}`, `{PROJECT_DISPLAY_NAME}`

### Phase Dependencies and Timing
- **Phase 1** (Requirements): 20s timeout, parallel processing enabled
- **Phase 2** (Test Strategy): 15s timeout, rule-based decisions
- **Phase 3** (Test Cases): 25s timeout, parallel file generation
- **Phase 4** (E2E Documentation): 25s timeout, runs parallel with Phase 3
- **Total Execution**: 90s (1.5 minutes) ultra-fast

### Output Structure
```
workflow_outputs/{COMPONENT}/{JIRA_KEY}/
├── phases/                        # Phase outputs
│   ├── test_requirements_output.yaml
│   ├── {JIRA_KEY}_test_strategy_output.yaml
│   └── {JIRA_KEY}_e2e_test_results.md
└── test_cases/                    # Generated test cases
    ├── {JIRA_KEY}_polarion.xml
    └── {JIRA_KEY}_test_case.md
```

### E2E Test Generation
- **File Pattern**: `{COMPONENT}_{JIRA_NUMBER}_{TEST_TYPE}.go`
- **Branch Pattern**: `ai-case-design-{JIRA_KEY}`
- **Repository**: Component-specific repository path
- **Auto-commit**: Disabled by default (configurable)

## Integration Points

### JIRA Integration
- **MCP Endpoint**: `jira-mcp-snowflake` 
- **Timeout**: 30s per query
- **Data Source**: Snowflake-backed JIRA data warehouse

### DeepWiki Integration  
- **MCP Endpoint**: `DeepWiki MCP`
- **Purpose**: Code change analysis and testing implications
- **Timeout**: 30s per query
- **Parallel Queries**: Up to 8 concurrent

### GitHub Integration
- **Repository Cloning**: Enabled for component analysis
- **Branch Management**: Automatic branch creation for generated tests
- **File Creation**: Direct file creation in e2e repositories

## Error Handling and Fallbacks

### Component Detection
- **Auto-detection**: From JIRA key prefix (`HIVE-*` � hive, `CCO-*` � cco)
- **Fallback**: Default to `hive` component if detection fails
- **Aliases**: Support multiple name variations per component

### Missing Configuration
- **Missing Rules**: Use fallback from `config/rules/hive-rules/`
- **Missing Templates**: Use base templates from `config/templates/`
- **Missing Agents**: Use universal agents with basic functionality

### Performance Optimization
- **Caching**: 5-minute TTL for DeepWiki queries, 30-minute TTL for repeated patterns
- **Sequential Processing**: Phase 3 → Phase 4 dependency chain
- **Resource Limits**: 8GB memory, 16 CPU cores maximum
- **Fast-fail**: Immediate stop on critical path failures
- **Query Optimization**: Batched DeepWiki calls with caching

## Development Workflow

### Adding New Components
1. Update `config/components.yaml` with new component definition
2. Create component-specific rules in `config/rules/{COMPONENT}-rules/`
3. Create `config/rules/{COMPONENT}-rules/component_rules_config.yaml` with agent mappings
4. Add component templates in `config/templates/` if needed
5. Add examples in `config/examples/{COMPONENT}-examples/`
6. Test with sample JIRA ticket

### Modifying Workflow
1. Update `config/test_case_generation_workflow.yaml` for timing/dependency changes
2. Modify agent configurations in `config/agents/` as needed
3. Update rules in component-specific directories
4. Update component detection rules in `config/components.yaml`
5. Test end-to-end workflow execution

### Custom Agent Development
1. Use existing agent configurations in `config/agents/` as templates
2. Define clear input/output contracts following template structure
3. Set appropriate timeouts and resource limits (60s per phase)
4. Include error handling and fallback strategies
5. Follow component-specific rules pattern in `config/rules/{COMPONENT}-rules/`

## Workflow Execution Protocol

### Critical Execution Requirements
- **NEVER skip phases**: All 4 phases must be executed in order
- **Dynamic variables**: Always use `{COMPONENT}`, `{JIRA_KEY}` for path resolution
- **Template compliance**: Strictly follow output templates from `config/templates/`
- **Component rules**: Load component-specific rules from `config/rules/{COMPONENT}-rules/component_rules_config.yaml`
- **Validation**: Verify each phase output before proceeding to next phase

### Agent Configuration Loading
Each agent follows this pattern:
1. Load agent config from `config/agents/{AGENT_NAME}.yaml`
2. Extract component from JIRA key prefix (HIVE-* → hive, CCO-* → cco)
3. Load component rules from `config/rules/{COMPONENT}-rules/component_rules_config.yaml`
4. Apply agent-specific rules listed in the `agent_rules` section
5. Use templates from `config/templates/` for output structure

### Output File Management
- **Requirements**: `workflow_outputs/{COMPONENT}/{JIRA_KEY}/
├── phases/                        # Phase outputstest_requirements_output.yaml`
- **Strategy**: `workflow_outputs/{COMPONENT}/{JIRA_KEY}/
├── phases/                        # Phase outputs{JIRA_KEY}_test_strategy_output.yaml`
- **Test Cases**: `workflow_outputs/{COMPONENT}/{JIRA_KEY}/test_cases/` (multiple files)
- **E2E Results**: `workflow_outputs/{COMPONENT}/{JIRA_KEY}/
├── phases/                        # Phase outputs{JIRA_KEY}_e2e_test_results.md`

## Available Agents

The system includes 10 specialized agents for different workflow phases:

### Core Workflow Agents
1. **requirements_gathering** - Extracts test requirements from JIRA tickets
2. **test_strategy_analysis** - Determines optimal test execution strategy
3. **test_case_generation** - Aggregates checkpoints and generates test scenarios
4. **e2e_test_generation** - Generates and integrates E2E test code into openshift-tests-private repository

### Specialized Agents
5. **e2e_code_generator** - Generates actual test code implementation
6. **e2e_validator_integrator** - Validates and integrates E2E tests
7. **environment_checker** - Validates test environment requirements
8. **test_result_analyzer** - Analyzes test execution results
9. **e2e_test_runner** - Executes E2E tests in target environments
10. **test-executor** - Comprehensive test execution orchestrator

### Agent Configuration Pattern
Each agent follows this structure:
- **Role Definition**: Specific responsibility and specialization
- **Task Steps**: Ordered execution steps with clear dependencies
- **Input Requirements**: Required parameters and data sources
- **Output Format**: Structured output following templates
- **Tool Configuration**: Available tools and MCP integrations
- **Dynamic Variables**: Runtime variable substitution (`{COMPONENT}`, `{JIRA_KEY}`)

## Important Notes

- **Execution Time**: Ultra-fast 90 seconds total execution time (50% improvement)
- **File Safety**: All file operations use safe paths with validation
- **Dynamic Variables**: All paths and configurations use runtime variable replacement
- **Security**: No sensitive data exposure in generated outputs
- **Validation**: All YAML outputs must be parseable and valid
- **Dependencies**: Strict dependency resolution between workflow phases
- **Component Focus**: Hive is the primary component, CCO is secondary, OCPBUGS auto-detected
- **Workflow Integrity**: NEVER skip phases - all 4 phases are mandatory for complete execution
- **Path Configuration**: E2E repository paths in `config/components.yaml` are user-configurable
- **MCP Integration**: Uses `jira-mcp-snowflake` for JIRA data and `DeepWiki MCP` for code analysis
- **Auto-Detection**: OCPBUGS tickets use smart component detection based on content analysis
- **Checklist Compliance**: Follow `prompts/check_list.md` for comprehensive execution validation