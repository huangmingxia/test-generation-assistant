# 🧪 Test Generation Assistant

## 🎯 Overview

AI-assisted testing

## 📋 Prerequisites

Before you can use this test generation workflow, you need to set up the following integrations:

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
### 4. Prepare Test Generation Rules
- **Purpose**: Configure rules to guide AI in generating realistic and executable test cases
- **Location**: `config/rules/`
- **Content**: Component-specific testing guidelines, validation criteria, and best practices

### 5. Configure GitHub CLI (gh)
- **Purpose**: Enable PR creation and GitHub repository operations
- **Installation**: 
  ```bash
  # macOS
  brew install gh
  
  # Linux/Windows
  # Follow: https://cli.github.com/manual/installation
  ```
  
### 6. Execution Method

**CRITICAL**: All workflows must be executed by reading agent YAML configurations directly.  

**Execution Pattern**:
1. Read `CLAUDE.md` for workflow instructions
2. Read specific agent YAML config (e.g., `config/agents/test_case_generation.yaml`)
3. Execute each step manually using available tools
4. Verify each step before proceeding

## 🚀  Workflows

### 1. Generate Test Cases
**Purpose**: Generate manual test cases from JIRA tickets  
**Agent Config**: `config/agents/test_case_generation.yaml`  
**Output**: Test cases in markdown format for Polarion integration  

### 2. Generate E2E Test Code
**Purpose**: Generate E2E test code based on existing test cases  
**Agent Config**: `config/agents/e2e_test_generation_openshift_private.yaml`  
**Output**: E2E test code integrated into openshift-tests-private repository  

### 2.1 Execute E2E Tests (Post-Generation)
**Purpose**: Execute generated E2E tests and capture results  
**Agent Config**: `config/agents/test-executor.yaml`  
**Output**: Test execution results and reports  

### 3. Submit E2E Pull Request 
**Purpose**: Create PR for generated E2E tests  
**Tools**: Git, GitHub CLI (gh)  
**Output**: Created pull request with E2E test code  
**Example**:  
```
> submit this PR

⏺ I'll submit the PR for the HIVE-2923 E2E test. Let me first check the current status and then 02:33 AMclaude-sonnet-4-20250514
  create the pull request.

⏺ Bash(git status)
  ⎿  On branch ai-case-design-HIVE-2923                                     
     Changes not staged for commit:
       (use "git add <file>..." to update what will be committed)
       (use "git restore <file>..." to discard changes in working directory)
        modified:   test/extended/cluster_operator/hive/hive.go

     no changes added to commit (use "git add" and/or "git commit -a")

⏺ Bash(git diff --name-only)
  ⎿  test/extended/cluster_operator/hive/hive.go

⏺ Bash(git add test/extended/cluster_operator/hive/hive.go)
  ⎿  (No content)

⏺ Bash(git commit -m "Add E2E test for HIVE-2923: AWS RequestID scrubbing in DNSZone

      Implement comprehensive E2E test to validate AWS RequestID scrubbing
      functionality in DNSZone controller to prevent thrashing behavior.

      Test validates:
      - DNSZone creation with invalid AWS credentials
      - RequestID (UUID) scrubbing from DNSError status conditions
      - Controller stability without excessive requeuing
      - Error message retention while removing RequestID

      🤖 Generated with [Claude Code](https://claude.ai/code)

      Co-Authored-By: Claude <noreply@anthropic.com>")
  ⎿  [ai-case-design-HIVE-2923 82db92ff0] Add E2E test for HIVE-2923: AWS RequestID scrubbing in DNSZone
      1 file changed, 123 insertions(+)

⏺ Bash(git push -u origin ai-case-design-HIVE-2923)
  ⎿  branch 'ai-case-design-HIVE-2923' set up to track 'origin/ai-case-design-HIVE-2923'.                          
     remote:
     remote: Create a pull request for 'ai-case-design-HIVE-2923' on GitHub by visiting:
     remote:      https://github.com/huangmingxia/openshift-tests-private/pull/new/ai-case-design-HIVE-2923
     remote:
     To github.com:huangmingxia/openshift-tests-private.git
      * [new branch]          ai-case-design-HIVE-2923 -> ai-case-design-HIVE-2923

⏺ Bash(gh pr create --title "Add E2E test for HIVE-2923: AWS RequestID scrubbing in DNSZone" --body "## Summary
      - Add comprehensive E2E test for HIVE-2923 to validate AWS RequestID scrubbing functionality
      - Test ensures DNSZone controller prevents thrashing due to changing RequestIDs in error messages
      - Validates both RequestID scrubbing and controller stability under error conditions

      ## Test Coverage
      - DNSZone creation with manageDNS enabled
      - Invalid AWS credentials to trigger authentication errors
      - RequestID (UUID) pattern detection and scrubbing validation
      - DNSZone controller stability monitoring
      - Error message content preservation while removing RequestIDs

      ## Test Plan
      - [x] Test compiles and follows existing Hive test patterns
      - [x] Uses platform-specific setup for AWS testing
      - [x] Includes proper cleanup and resource management
      - [x] Validates HIVE-2923 bug fix implementation

      🤖 Generated with [Claude Code](https://claude.ai/code)")
  ⎿  Error: (eval):1: command not found: gh

⏺ The gh CLI tool is not available. Let me provide the PR creation URL that was shown in the git02:33 AMclaude-sonnet-4-20250514
   push output:

  The pull request can be created manually by visiting:
  https://github.com/huangmingxia/openshift-tests-private/pull/new/ai-case-design-HIVE-2923
```

### 4. Write test case to Polarion [In Progress]
**Purpose**: Update Polarion with test case information  
**Tools**: Polarion API integration  
**Output**: Updated Polarion test cases  

### 5. Update JIRA QE Comment [In Progress]


### Workflow Dependencies
- Workflow 2.1 requires completion of Workflow 2
- Workflow 3 requires completion of Workflow 2 (and optionally 2.1)
- Workflow 4 requires completion of Workflow 1
- Workflow 5 can be executed after any workflow completion

## 🚀 Claude Code Usage

### Quick Start

1. **Setup Prerequisites** (see above sections):
   - Install Claude Code
   - Configure JIRA MCP access
   - Configure DeepWiki MCP

2. **Open Claude Code**:
   ```bash
   claude
   ```

3. **Navigate to project directory**:
   ```bash
   cd /path/to/test-generation-assistant
   ```

4. **Execute workflows directly**:
   ```
   # Generate test cases
   "Create test case for HIVE-2883"
   
   # Generate E2E code
   "Generate E2E code for HIVE-2883"
   
   # Execute E2E tests
   "Run E2E tests for HIVE-2883"
   
   # Create PR
   "Create PR for HIVE-2883 E2E tests"
   
   # Update Polarion
   "Update Polarion with HIVE-2883 test cases"
   
   # Add QE comment
   "Add QE comment to HIVE-2883"
   ```

### How It Works

1. **Claude reads CLAUDE.md** - Gets workflow instructions
2. **Reads agent YAML configs** - Gets specific execution steps  
3. **Executes steps** - Uses available tools (JIRA MCP, WebFetch, Bash, etc.)
4. **Verifies outputs** - Ensures each step completes successfully
5. **Generates artifacts** - Creates test cases, E2E code, PRs, etc.

### No API Server Needed
- Everything runs through Claude Code directly
- No need for separate microservice or Docker containers
- All workflows executed via natural language commands
- Real-time interaction and feedback

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

- **[CLAUDE.md](CLAUDE.md)** - Complete workflow instructions and execution guide
- **[prompts/STARTUP_CHECKLIST.md](prompts/STARTUP_CHECKLIST.md)** - Mandatory execution checklist
- **[config/agents/](config/agents/)** - Individual agent configurations
- **[config/templates/](config/templates/)** - Output templates for all workflows
