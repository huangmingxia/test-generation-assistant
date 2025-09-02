# Test Case Generation Workflow

An AI-powered system for generating comprehensive test cases for OpenShift components, specifically Hive and Cloud Credential Operator (CCO). The system uses a 4-phase workflow engine to automatically generate test documentation and E2E test code from JIRA tickets.

## 📋 Prerequisites

Before you can use this test case generation workflow, you need to set up the following integrations:

### 1. Install Claude Code 
- **Claude Code access**: Follow [Claude Code Install Instructions](https://docs.google.com/document/d/1eNARy9CI28o09E7Foq01e5WD5MvEj3LSBnXqFcprxjo/edit?usp=drivesdk)

### 2. JIRA MCP Access
- **Apply for jira-mcp-snowflake token**: Refer to [Jira-MCP-Snowflake-Token-Guide](https://docs.google.com/document/d/1pg6TkwezhIahppp5k0md0Zx0CC-4f_RWQHaH9cTl1Mo/edit?tab=t.0#heading=h.xyjdx8nsdjql)
 
- **Config jira-mcp-snowflake mcp server**:
  ```bash
  claude mcp add jira-mcp-snowflake https://jira-mcp-snowflake.mcp-playground-poc.devshift.net/sse --transport sse -H "X-Snowflake-Token: your_token_here"
  ```
### 3. DeepWiki MCP Connection
  ```bash
  claude mcp add -s user -t http deepwiki https://mcp.deepwiki.com/mcp
  ```
### 4. 🔧 Configuration Guide

For other components wanting to use this workflow, see the comprehensive [User Configuration Guide](USER_CONFIGURATION_GUIDE.md) which includes:



## 🚀 Quick Start

1. **Configure your environment** by updating paths in `config/components.yaml`
For other components wanting to use this workflow, see the comprehensive [User Configuration Guide](USER_CONFIGURATION_GUIDE.md) which includes:

2. **Provide a JIRA ticket key** (e.g., `HIVE-2883`, `CCO-1234`)
3. **Execute the 4-phase workflow** following the prompts
4. **Total execution time**: 4 minutes (240 seconds) standard mode, 90 seconds (1.5 minutes) fast mode

## 📋 Overview

### Core Features

- **4-Phase Workflow Engine**: Sequential execution with parallel optimization
- **Multi-Component Support**: OpenShift Hive (`HIVE-*`) and Cloud Credential Operator (`CCO-*`)
- **Configuration-Driven**: All behavior controlled through YAML configurations
- **Agent-Based Architecture**: 10 specialized agents for different aspects of test generation
- **Template-Driven Output**: Standardized formats with component-specific variations
- **Dual Execution Modes**: 4-minute standard mode or 90-second fast mode with parallel processing

### Supported Components

| Component | JIRA Prefix | Repository |
|-----------|-------------|------------|
| **OpenShift Hive** | `HIVE-*` | `github.com/openshift/hive` |
| **Cloud Credential Operator** | `CCO-*` | `github.com/openshift/cloud-credential-operator` |

## 📁 Project Structure

```
test_case_context/
├── config/                        # Configuration directory
│   ├── agents/                    # Agent configuration files
│   │   ├── requirements_gathering.yaml
│   │   ├── test_strategy_analysis.yaml
│   │   ├── test_case_generation.yaml
│   │   ├── e2e_test_generation.yaml
│   │   ├── e2e_test_runner.yaml
│   │   ├── e2e_validator_integrator.yaml
│   │   ├── environment_checker.yaml
│   │   └── test_result_analyzer.yaml
│   ├── components.yaml            # Component definitions and settings
│   ├── test_case_generation_workflow.yaml # Test case generation workflow
│   ├── test_execution_workflow.yaml # Test execution workflow
│   ├── rules/                     # Component-specific rules and guidelines
│   │   ├── hive-rules/            # Hive component rules
│   │   │   ├── component_rules_config.yaml
│   │   │   ├── e2e_test_case_guidelines.md
│   │   │   ├── e2e_mandatory_steps.yaml
│   │   │   └── test_type_selection_criteria.yaml
│   │   ├── cco-rules/             # CCO component rules
│   │   │   ├── component_rules_config.yaml
│   │   │   ├── e2e_test_case_guidelines.md
│   │   │   └── test_type_selection_criteria.yaml
│   │   └── common/                # Common rules and patterns
│   │       └── universal_e2e_pattern_learning.yaml
│   ├── templates/                 # Output templates
│   │   ├── test_requirements_template.yaml
│   │   ├── test_strategy_template.yaml
│   │   ├── test_cases_template.yaml
│   │   ├── test_cases_polarion_template.yaml
│   │   └── e2e_test_results_template.md
│   └── examples/                  # Example files and outputs
│       ├── hive-examples/         # Hive component examples
│       │   ├── hive-comprehensive-test-results-example.md
│       │   ├── hive-e2e-test-execution-results-example.md
│       │   ├── hive-test-correction-report-example.md
│       │   └── hive-test-script-example.sh
│       └── cco-examples/          # CCO component examples
│           └── cco-polarion-test-case-example-simplified.xml
├── prompts/                       # Workflow execution guides
│   ├── execution_guide.md        # Comprehensive execution guide (4-min + 90-sec modes)
│   ├── workflow_guide.md         # Complete workflow guide (generation + execution)
│   ├── coordinator_execution_guide.md # E2E coordinator execution (advanced)
│   └── check_list.md             # Comprehensive execution checklist
├── workflow_outputs/              # Generated test outputs
│   ├── hive/                      # Hive component outputs
│   │   └── {JIRA_KEY}/           # e.g., HIVE-2040
│   │       ├── phases/            # Phase outputs (requirements, strategy)
│   │       ├── test_cases/        # Generated test cases
│   │       └── test_execution_results/ # Test execution results
│   └── cco/                       # CCO component outputs
│       └── {JIRA_KEY}/           # e.g., CCO-681
│           ├── phases/            # Phase outputs
│           ├── test_cases/        # Generated test cases
│           └── test_execution_results/ # Test execution results
├── USER_CONFIGURATION_GUIDE.md    # Complete configuration guide for new components
├── CLAUDE.md                      # Project instructions for Claude Code
└── README.md                      # This file
```

## 🚀 Quick Start

### 1. Configure Components

Edit the `config/components.yaml` file to configure your components:

```yaml
components:
  hive:
    name: "Hive"
    repository: "openshift/hive"
    rules_path: "config/rules/hive-rules/"
    examples_path: "config/examples/hive-examples/"
```

### 2. Run Workflow

```bash
# Input JIRA issue key
JIRA_KEY="HIVE-2040"
COMPONENT="hive"

# Execute workflow
# System will automatically execute the following phases:
# 1. Requirements gathering
# 2. Test strategy analysis
# 3. Test case generation
# 4. E2E test generation (if needed)
```

## 🏗️ Architecture

### 4-Phase Workflow Engine

```
Phase 1 (60s)  → Requirements Gathering from JIRA
Phase 2 (60s)  → Test Strategy Analysis (depends on Phase 1)
Phase 3 & 4    → Parallel execution (150s total)
   ├── Phase 3: Test Case Generation
   └── Phase 4: E2E Test Generation
```

**Total Execution**: 240 seconds (4 minutes) standard mode, 90 seconds (1.5 minutes) fast mode with parallel processing

### Agent Architecture

#### Core Workflow Agents
1. **requirements_gathering** - Extracts test requirements from JIRA tickets
2. **test_strategy_analysis** - Determines optimal test execution strategy
3. **test_case_generation** - Aggregates checkpoints and generates test scenarios
4. **e2e_test_generation** - Generates and integrates E2E test code into openshift-tests-private repository

#### Specialized Agents
5. **e2e_code_generator** - Generates actual test code implementation
6. **e2e_validator_integrator** - Validates and integrates E2E tests
7. **environment_checker** - Validates test environment requirements
8. **test_result_analyzer** - Analyzes test execution results
9. **e2e_test_runner** - Executes E2E tests in target environments
10. **test-executor** - Comprehensive test execution orchestrator

## 🔄 Workflow Phases

### Phase 1: Requirements Gathering (60s)
- **Input**: JIRA issue key
- **Process**: Extract requirements using `jira-mcp-snowflake`
- **Output**: `test_requirements_output.yaml`

### Phase 2: Test Strategy Analysis (60s)
- **Input**: Phase 1 output + component rules
- **Process**: Determine test execution strategy
- **Output**: `{JIRA_KEY}_test_strategy_output.yaml`

### Phase 3: Test Case Generation (150s, parallel)
- **Input**: Phase 1 & 2 outputs + templates
- **Process**: Generate Polarion XML and Markdown test cases
- **Output**: `_polarion.xml`, `_test_case.md`

### Phase 4: E2E Test Generation (150s, parallel)
- **Input**: Phase 1 & 2 outputs + E2E patterns
- **Process**: Generate E2E test documentation with code integration
- **Output**: `_e2e_test_results.md` + actual test code files

### Output Structure

```
workflow_outputs/{COMPONENT}/{JIRA_KEY}/
├── phases/
│   ├── test_requirements_output.yaml
│   ├── {JIRA_KEY}_test_strategy_output.yaml
│   └── {JIRA_KEY}_e2e_test_results.md
└── test_cases/
    ├── {JIRA_KEY}_polarion.xml
    └── {JIRA_KEY}_test_case.md
```

## ⚙️ Configuration

### Component Configuration (`config/components.yaml`)

```yaml
components:
  hive:
    name: "Hive"
    repository: "openshift/hive"
    rules_path: "config/rules/hive-rules/"
    examples_path: "config/examples/hive-examples/"
    
  cco:
    name: "Cloud Credential Operator"
    repository: "openshift/cloud-credential-operator"
    rules_path: "config/rules/cco-rules/"
    examples_path: "config/examples/cco-examples/"

component_detection:
  jira_to_component:
    HIVE: "hive"
    CCO: "cco"
```

### Workflow Configuration (`config/test_case_generation_workflow.yaml`)

- **Target execution time**: 240 seconds (4 minutes) standard mode, 90 seconds (1.5 minutes) fast mode
- **Parallel execution**: Phase 3 & 4 run concurrently
- **Ultra-fast mode**: Aggressive optimization enabled
- **Resource limits**: 2GB memory (fast mode), 4GB memory (standard mode), 4-8 CPU cores

## 🛠️ Advanced Features

### Component-Specific Rules

Each component has its own rule system:

```yaml
# config/rules/{COMPONENT}-rules/component_rules_config.yaml
agent_rules:
  requirements_gathering:
    - "hive_day1_test_case_rules.yaml"
    - "test_type_selection_criteria.yaml"
  test_strategy_analysis:
    - "hive_operator_test_rules.yaml"
    - "e2e_test_case_guidelines.yaml"
```

### E2E Code Generation

- **Automatic branch creation**: `ai-case-design-{JIRA_KEY}`
- **Code integration**: Direct file updates in E2E repository
- **Validation**: Syntax checking and format validation
- **Test data creation**: YAML templates and configuration files

### Performance Optimization

- **Parallel execution**: Phase 3 & 4 run simultaneously
- **Caching**: 5-minute TTL for component rules, 10-minute TTL for DeepWiki queries
- **Resource optimization**: 8-core CPU allocation for parallel processing
- **Fast-fail**: Immediate stop on critical path failures

## 🔗 Integration Points

### JIRA Integration
- **MCP Endpoint**: `jira-mcp-snowflake`
- **Data Source**: Snowflake-backed JIRA data warehouse
- **Timeout**: 30 seconds per query

### DeepWiki Integration
- **MCP Endpoint**: `DeepWiki MCP`
- **Purpose**: Code change analysis and testing implications
- **Parallel Queries**: Up to 8 concurrent connections

### GitHub Integration
- **Repository Management**: Automatic branch creation
- **File Operations**: Direct file creation in E2E repositories
- **Version Control**: Git integration for test code management

## 📋 Execution Checklist

### Pre-Execution Requirements
- [ ] JIRA issue exists and is accessible
- [ ] E2E repository paths are accessible and writable
- [ ] Component configuration loaded
- [ ] Dynamic variables initialized

### Phase Completion Markers

#### ✅ Phase 1 Complete
- [ ] JIRA issue data retrieved successfully
- [ ] `test_requirements_output.yaml` generated
- [ ] File size > 30 lines
- [ ] Execution time ≤ 60 seconds

#### ✅ Phase 2 Complete
- [ ] Phase 1 output loaded successfully
- [ ] Component rules applied
- [ ] `test_strategy_output.yaml` generated
- [ ] E2E decision clear (YES/NO)
- [ ] Execution time ≤ 60 seconds

#### ✅ Phase 3&4 Parallel Complete
- [ ] Both phases started simultaneously
- [ ] Test cases generated (Phase 3)
- [ ] E2E documentation generated (Phase 4)
- [ ] Both complete within 150 seconds
- [ ] No inter-phase waiting time

## 🚨 Troubleshooting

### Common Issues

1. **Timeout Errors**: Check network connectivity to JIRA/DeepWiki
2. **Missing Components**: Verify component detection in `config/components.yaml`
3. **Rule Loading Failures**: Check rule file paths and permissions
4. **Parallel Execution Issues**: Verify dependency resolution

### Performance Tuning

- **Memory**: Increase from 2GB (fast mode) or 4GB (standard mode) if processing large JIRA tickets
- **CPU**: Scale up from 4-8 cores for faster parallel execution
- **Timeouts**: Adjust phase timeouts based on network conditions
- **Caching**: Tune TTL values for better performance

## 📊 Performance Metrics

### Target Performance
- **Phase 1**: 45-60 seconds (JIRA data retrieval)
- **Phase 2**: 30-45 seconds (Strategy analysis)
- **Phase 3&4**: 90-150 seconds (Parallel execution)
- **Total**: 165-255 seconds (15-45 second buffer)

### Optimization Features
- **50% speed improvement** through parallel execution
- **Aggressive caching** for repeated operations
- **Resource prioritization** for critical path phases
- **Error fast-fail** to avoid time waste

## 📝 Development

### Adding New Components

1. **Follow the [User Configuration Guide](USER_CONFIGURATION_GUIDE.md)** for detailed step-by-step instructions
2. Update `config/components.yaml` with component definition
3. Create component-specific rules in `config/rules/{COMPONENT}-rules/`
4. Add examples in `config/examples/{COMPONENT}-examples/`
5. Test with sample JIRA ticket
5. 📖 参考 [USER_CONFIGURATION_GUIDE.md](USER_CONFIGURATION_GUIDE.md) 获取详细的组件配置说明

### Modifying Workflow

1. Update `config/test_case_generation_workflow.yaml`
2. Modify agent configurations in `config/agents/`
3. Update component rules as needed
4. Test end-to-end execution

### Custom Agent Development

1. Use existing configurations as templates
2. Define clear input/output contracts
3. Set appropriate timeouts (60s per phase)
4. Include error handling and fallback strategies

## 🔒 Security & Safety

- **No sensitive data exposure** in generated outputs
- **Safe file operations** with path validation
- **Component isolation** through rule-based access
- **Audit trail** through comprehensive logging

## 📚 Related Documentation

- `USER_CONFIGURATION_GUIDE.md` - Complete configuration guide for new components
- `prompts/execution_guide.md` - Comprehensive execution guide (4-minute standard + 90-second fast mode)
- `prompts/workflow_guide.md` - Complete workflow guide (test case generation + test execution)
- `prompts/coordinator_execution_guide.md` - E2E coordinator execution (advanced usage)
- `prompts/check_list.md` - Comprehensive execution checklist
- `CLAUDE.md` - Project instructions for Claude Code

## 🤝 Contributing

1. Follow existing configuration patterns
2. Test with sample JIRA tickets
3. Update documentation for new features
4. Maintain backward compatibility
5. Ensure all phases execute within time limit# test-generation-assistant
