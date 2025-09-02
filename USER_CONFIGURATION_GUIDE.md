# User Configuration Guide

This guide provides step-by-step instructions for configuring and using the Test Case Generation Workflow for your OpenShift component.

## 📋 Prerequisites

Before configuring the workflow for your component, ensure you have:

1. **Claude Code access** - Follow [Claude Code Install Instructions](https://docs.google.com/document/d/1eNARy9CI28o09E7Foq01e5WD5MvEj3LSBnXqFcprxjo/edit?usp=drivesdk)

2. **JIRA MCP Access** - Apply for jira-mcp-snowflake token:
   - Refer to [Jira-MCP-Snowflake-Token-Guide](https://docs.google.com/document/d/1pg6TkwezhIahppp5k0md0Zx0CC-4f_RWQHaH9cTl1Mo/edit?tab=t.0#heading=h.xyjdx8nsdjql)
   - Configure the MCP server:
   ```bash
   claude mcp add jira-mcp-snowflake https://jira-mcp-snowflake.mcp-playground-poc.devshift.net/sse --transport sse -H "X-Snowflake-Token: your_token_here"
   ```

3. **DeepWiki MCP Connection**:
   ```bash
   claude mcp add -s user -t http deepwiki https://mcp.deepwiki.com/mcp
   ```

## 🚀 Quick Setup for New Components

### Step 1: Component Configuration

Edit `config/components.yaml` to add your component:

```yaml
# Add your component to the components section
components:
  your-component:
    name: "Your Component Name"
    repository: "openshift/your-component"
    rules_path: "config/rules/your-component-rules/"
    examples_path: "config/examples/your-component-examples/"

# Add component detection mapping
component_detection:
  jira_to_component:
    YOUR: "your-component"  # Replace YOUR with your JIRA prefix
    
  auto_detection:
    component_field_mapping:
      "Your Component Name": "your-component"
      "your-component": "your-component"
```

### Step 2: Create Component Rules Directory

Create the following directory structure:

```bash
mkdir -p config/rules/your-component-rules/
mkdir -p config/examples/your-component-examples/
```

### Step 3: Create Component Rules

Create these essential rule files in `config/rules/your-component-rules/`:

#### component_rules_config.yaml
```yaml
agent_rules:
  requirements_gathering:
    - "your_component_test_case_rules.yaml"
    - "test_type_selection_criteria.yaml"
  test_strategy_analysis:
    - "your_component_operator_test_rules.yaml"
    - "e2e_test_case_guidelines.yaml"
  test_case_generation:
    - "your_component_test_patterns.yaml"
    - "polarion_template_rules.yaml"
  e2e_test_generation:
    - "your_component_e2e_patterns.yaml"
    - "e2e_mandatory_steps.yaml"
```

#### test_type_selection_criteria.yaml
```yaml
test_types:
  manual:
    criteria:
      - "UI interactions required"
      - "Complex user workflows"
      - "Non-automatable scenarios"
  automated:
    criteria:
      - "API testing"
      - "CLI operations"
      - "Configuration validation"
  e2e:
    criteria:
      - "Full feature lifecycle"
      - "Multi-component integration"
      - "End-to-end user scenarios"
```

#### e2e_test_case_guidelines.md
```markdown
# E2E Test Case Guidelines for Your Component

## Test Structure
- Use descriptive test names
- Include proper setup and teardown
- Add comprehensive assertions
- Follow component-specific patterns

## Mandatory Steps
1. Environment validation
2. Component deployment verification
3. Feature testing
4. Cleanup procedures

## Best Practices
- Use existing test utilities
- Follow naming conventions
- Include proper error handling
- Add meaningful comments
```

### Step 4: Create Example Files

Add example files in `config/examples/your-component-examples/`:

#### your-component-polarion-test-case-example-simplified.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<testcase>
  <title>Your Component Test Case Example</title>
  <description>Example test case for your component</description>
  <teststeps>
    <step>
      <step_number>1</step_number>
      <step_description>Setup test environment</step_description>
      <expected_result>Environment is ready for testing</expected_result>
    </step>
  </teststeps>
</testcase>
```

#### your-component-test-results-example.md
```markdown
# Test Results Example

## Test Execution Summary
- **Test Case**: Your Component Feature Test
- **Status**: PASSED
- **Duration**: 2 minutes
- **Environment**: OCP 4.15

## Detailed Results
1. ✅ Environment setup completed
2. ✅ Component deployment successful
3. ✅ Feature functionality verified
4. ✅ Cleanup completed

## Issues Found
None

## Recommendations
- Test case is ready for production use
```

### Step 5: Update Agent Configurations

Modify agent configurations in `config/agents/` to include your component:

#### requirements_gathering.yaml
```yaml
# Add your component to the supported components list
supported_components:
  - "hive"
  - "cco"
  - "your-component"  # Add this line
```

## 🔧 Configuration Details

### Component-Specific Settings

#### Basic Configuration
- **name**: Display name for your component
- **repository**: GitHub repository path (e.g., "openshift/your-component")
- **rules_path**: Path to component-specific rules directory
- **examples_path**: Path to component-specific examples directory

#### Component Detection
- **jira_to_component**: Maps JIRA ticket prefixes to component names
- **auto_detection**: Enables automatic component detection from JIRA component field
- **component_field_mapping**: Maps JIRA component field values to component names

### Workflow Configuration

The workflow automatically detects your component based on:

1. **JIRA Ticket Prefix**: `YOUR-1234` (replace YOUR with your component prefix)
2. **JIRA Component Field**: The component field in the JIRA ticket
3. **Auto-detection**: Falls back to auto-detection if prefix mapping fails

Ensure your JIRA tickets have the correct component field set to enable automatic detection.

### E2E Test Integration

When E2E tests are generated, they will be placed in the default openshift-tests-private repository location. The system uses the global `kubeconfig_path` setting for cluster access.

**Note**: E2E test generation requires:
- Valid kubeconfig path in the global configuration
- Write access to the openshift-tests-private repository
- Proper component rules configuration for E2E patterns

## 🚀 Usage Instructions

### 1. Basic Execution

```bash
# Set your JIRA ticket key
JIRA_KEY="YOUR-1234"

# The system will automatically:
# 1. Detect your component from JIRA prefix
# 2. Load component-specific rules
# 3. Execute the 4-phase workflow
# 4. Generate test cases and E2E tests
```

### 2. Expected Output

After execution, you'll find outputs in:
```
workflow_outputs/your-component/YOUR-1234/
├── phases/
│   ├── test_requirements_output.yaml
│   ├── YOUR-1234_test_strategy_output.yaml
│   └── YOUR-1234_e2e_test_results.md
└── test_cases/
    ├── YOUR-1234_polarion.xml
    └── YOUR-1234_test_case.md
```

### 3. E2E Test Integration

If E2E tests are generated:
- A new branch `ai-case-design-YOUR-1234` will be created
- Test files will be added to your component's E2E directory
- Integration instructions will be provided in the output

## 🔍 Troubleshooting

### Common Issues

1. **Component Not Detected**
   - Verify JIRA ticket prefix matches component configuration
   - Check `config/components.yaml` for correct component name

2. **Rules Not Loading**
   - Ensure all required rule files exist in `config/rules/your-component-rules/`
   - Check file permissions and paths

3. **E2E Repository Access**
   - Verify write access to openshift-tests-private
   - Check local repository path in component configuration

4. **Test Generation Failures**
   - Review component rules for completeness
   - Check example files for proper formatting
   - Verify JIRA ticket contains sufficient information

### Performance Optimization

- **Execution Time**: Target 4.5 minutes total
- **Memory Usage**: 1GB recommended
- **CPU**: 8 cores for parallel processing

## 📋 Validation Checklist

Before using your component configuration:

- [ ] Component added to `config/components.yaml`
- [ ] All required rule files created
- [ ] Example files provided
- [ ] E2E repository path configured
- [ ] JIRA ticket prefix defined
- [ ] Test with sample JIRA ticket
- [ ] Verify output generation
- [ ] Check E2E test integration (if applicable)

## 🔄 Maintenance

### Regular Updates

1. **Update Rules**: Modify component rules based on testing patterns
2. **Add Examples**: Include new example files for different test scenarios
3. **Review Outputs**: Analyze generated test cases for quality
4. **Optimize Performance**: Tune configuration for faster execution

### Version Compatibility

- Keep component rules compatible with workflow updates
- Test configuration changes with sample tickets
- Maintain backward compatibility when possible

## 📚 Additional Resources

- [Main README.md](README.md) - Complete workflow documentation
- [Execution Guide](prompts/execution_guide.md) - Comprehensive execution guide (4-minute standard + 90-second fast modes)
- [Check List](prompts/check_list.md) - Comprehensive execution checklist
- [CLAUDE.md](CLAUDE.md) - Project instructions for Claude Code
