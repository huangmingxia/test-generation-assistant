# Startup Checklist

## Core Configuration Validation
- [ ] Read `config/components.yaml` - Component definitions and settings
- [ ] Validate `config/test_case_generation_workflow.yaml` - Test case generation workflow
- [ ] Validate `config/test_execution_workflow.yaml` - Test execution workflow
- [ ] Load component-specific rules from `config/rules/{COMPONENT}-rules/`
- [ ] Initialize component mapping configuration

## Project Structure Validation
- [ ] Check `config/agents/` directory - All 9 agent configurations present
- [ ] Verify `config/rules/` directory structure - Component-specific rules
- [ ] Confirm `config/templates/` directory - 5 output templates available
- [ ] Validate `config/examples/` directory - Component examples available
- [ ] Check `prompts/` directory - Workflow guides and checklist files

## Environment Preparation
- [ ] Validate working directory structure
- [ ] Check necessary file existence and permissions
- [ ] Confirm e2e repository paths are accessible (from component rules)
- [ ] Initialize logging for test generation and execution

## Workflow Selection
- [ ] Test case generation request
- [ ] E2E test execution request
- [ ] E2E code PR submission request (In Progress)
- [ ] Polarion integration request (In Progress)

## Context Optimization
- [ ] Apply component-specific context filtering
- [ ] Select relevant JIRA issue details
- [ ] Load component-specific rules and patterns
- [ ] Set component context variables (`{COMPONENT}`, `{JIRA_KEY}`)

## Agent Loading
- [ ] Load `requirements_gathering` agent
- [ ] Load `test_strategy_analysis` agent
- [ ] Load `test_case_generation` agent
- [ ] Load `e2e_test_generation` agent
- [ ] Load `environment_checker` agent
- [ ] Load `test-executor` agent
- [ ] Load `test_result_analyzer` agent

## Component-Specific Rules
- [ ] Load component configuration (hive/cco)
- [ ] Apply component-specific test rules
- [ ] Select component templates
- [ ] Validate component-specific E2E patterns

## Documentation References
- [ ] Review [TEST_CASE_GENERATION_WORKFLOW.md](TEST_CASE_GENERATION_WORKFLOW.md) for test case generation
- [ ] Review [TEST_EXECUTION_WORKFLOW.md](TEST_EXECUTION_WORKFLOW.md) for test execution
- [ ] Review [CLAUDE.md](../CLAUDE.md) for Claude Code instructions
- [ ] Review [USER_CONFIGURATION_GUIDE.md](USER_CONFIGURATION_GUIDE.md) for component setup

## Configuration Files
- [ ] Verify `config/test_case_generation_workflow.yaml` exists and is valid
- [ ] Verify `config/test_execution_workflow.yaml` exists and is valid
- [ ] Verify `config/agents/` directory contains all required agent configs
- [ ] Verify `config/rules/{COMPONENT}-rules/` directory exists for target component

## Final Validation
- [ ] All configuration files loaded successfully
- [ ] Component-specific rules applied
- [ ] Agent configurations validated
- [ ] Template files accessible
- [ ] Output directory structure ready
- [ ] E2E repository paths configured and accessible

## Ready to Execute
- [ ] User request clearly understood
- [ ] Appropriate workflow selected
- [ ] All prerequisites met
- [ ] System ready for execution
