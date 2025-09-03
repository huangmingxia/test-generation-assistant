# 🧪 Test Generation Assistant

## 🎯 Overview

This system generates comprehensive test cases from JIRA tickets and executes E2E tests for OpenShift Hive (`HIVE-*`) and Cloud Credential Operator (`CCO-*`) components.

**Current Status**: 4 separate workflows that require QE review and validation between stages.

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

For other components wanting to use this workflow, see the [User Configuration Guide](prompts/USER_CONFIGURATION_GUIDE.md)

## 🚀 Available Workflows

### 1. Generate Test Cases, E2E test cases
**Purpose**: Generate test cases from JIRA tickets using 4-phase AI engine

**Agents Used**:
- `requirements_gathering` - Extract JIRA requirements
- `test_strategy_analysis` - Determine test strategy  
- `test_case_generation` - Generate test scenarios
- `e2e_test_generation` - Create E2E test code

**Output**: Polarion XML + Markdown test cases + E2E test code

**Documentation**: [prompts/TEST_CASE_GENERATION_WORKFLOW.md](prompts/TEST_CASE_GENERATION_WORKFLOW.md)

### 2. Execute E2E Tests
**Purpose**: Execute generated E2E tests and analyze results

**Agents Used**:
- `environment_checker` - Verify test environment and cluster connectivity
- `test-executor` - Execute E2E tests and analyze results (integrated agent)
- `test_result_analyzer` - Analyze test results (optional, as test-executor handles this)

**Documentation**: [TEST_EXECUTION_WORKFLOW.md](TEST_EXECUTION_WORKFLOW.md)

### 3. Submit E2E Code PR
**Status**: In Progress

### 4. Write Test Cases to Polarion  
**Status**: In Progress

### Supported Components
| Component | JIRA Prefix | Repository |
|-----------|-------------|------------|
| **Hive** | `HIVE-*` | `github.com/openshift/hive` |
| **Cloud Credential Operator** | `CCO-*` | `github.com/openshift/cloud-credential-operator` |

## 📝 Adding New Components

1. **Update** `config/components.yaml` with component definition
2. **Create** component-specific rules in `config/rules/{COMPONENT}-rules/`
3. **Add** examples in `config/examples/{COMPONENT}-examples/`
4. **Test** with sample JIRA ticket

## 🔗 Integrations

### JIRA Integration
- **MCP Endpoint**: `jira-mcp-snowflake`
- **Purpose**: Extract ticket requirements and details

### DeepWiki Integration
- **MCP Endpoint**: `DeepWiki MCP`
- **Purpose**: Code change analysis and testing implications

### GitHub Integration
- **Repository Management**: Automatic branch creation
- **File Operations**: Direct file creation in E2E repositories

## 📚 Documentation

- **[prompts/TEST_CASE_GENERATION_WORKFLOW.md](prompts/TEST_CASE_GENERATION_WORKFLOW.md)** - Complete test case generation guide
- **[prompts/TEST_EXECUTION_WORKFLOW.md](prompts/TEST_EXECUTION_WORKFLOW.md)** - E2E test execution guide
- **[prompts/USER_CONFIGURATION_GUIDE.md](prompts/USER_CONFIGURATION_GUIDE.md)** - Component setup guide
- **[CLAUDE.md](CLAUDE.md)** - Claude Code instructions

## 🤝 Contributing
1. Follow existing configuration patterns
2. Test with sample JIRA tickets
3. Update documentation for new features
4. Maintain backward compatibility

---

**Built for OpenShift testing automation**
