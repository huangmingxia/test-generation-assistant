# User Configuration Guide

This guide provides step-by-step instructions for configuring and using the Test Case Generation Workflow for your OpenShift component.

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
  test_strategy_analysis:
    - "your_component_operator_test_rules.yaml"
  test_case_generation:
    - "your_component_test_patterns.yaml"
  e2e_test_generation:
    - "your_component_e2e_patterns.yaml"
```


### Step 4: Create Example Files

Add example files in `config/examples/your-component-examples/`:
#### 1. your-component-polarion-test-case-example-simplified.xml
#### 2. ...

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
- [Execution Guide](prompts/execution_guide.md) - Comprehensive execution guide 
- [Check List](prompts/check_list.md) - Comprehensive execution checklist
- [CLAUDE.md](CLAUDE.md) - Project instructions for Claude Code
