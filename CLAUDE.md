# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 🚨 MANDATORY STARTUP PROTOCOL 🚨

**CRITICAL**: Before executing ANY user request, Claude MUST:
1. **ALWAYS read this CLAUDE.md file first** - This is mandatory, no exceptions
2. **Execute startup checklist** as defined in the workflow config (`startup_checklist: "prompts/STARTUP_CHECKLIST.md"`)
3. **Identify workflow type** based on user input (see Workflow Selection below)
4. **Use Task tool to call specialized agents** - Never execute workflows directly
5. **Follow agent configurations strictly** from `config/agents/`
6. **Adhere to templates exactly** from `config/templates/`
7. **🚨 MANDATORY: Respect Phase 4 conditional execution** - ONLY execute Phase 4 if `e2e_decision.required == YES`

**If you skip this protocol, the output will be incorrect and not follow the project's standards.**

## 🚨 CRITICAL EXECUTION RULE: Phase 4 Conditional Logic

**MANDATORY**: Phase 4 (`e2e_test_generation`) is **CONDITIONALLY EXECUTED**:
- ✅ **Execute Phase 4** ONLY if Phase 2 outputs `e2e_decision.required == YES`
- ❌ **SKIP Phase 4 COMPLETELY** if Phase 2 outputs `e2e_decision.required == NO`
- 📝 **NO documentation generation** when Phase 4 is skipped
- ⚙️ **Configuration Rule**: `execute_only_if: "e2e_decision.required == YES"` (line 99 in workflow config)

## Overview

This is the **Test Generation Assistant** - an AI-powered system for generating test cases and executing E2E tests for Hive and Cloud Credential Operator (CCO) now.

**Current Status**: The system currently has 4 separate workflows that are not merged together because each stage requires QE review and validation before proceeding to the next phase. 

## ⚡ Workflow Execution Protocol

**BEFORE processing any user request, Claude MUST:**
1. Read `prompts/STARTUP_CHECKLIST.md` for startup checklist validation
2. Follow the exact sequence defined in the workflow configuration
3. Use Task tool to call specialized agents - NEVER execute directly

## Workflow Selection

Based on user input, Claude should execute different workflows:

### 1. Generate Test Cases (4-Phase Workflow)
**When user asks to generate test cases:**
- Execute the complete 4-phase test case generation workflow, follow [prompts/TEST_CASE_GENERATION_WORKFLOW.md](prompts/TEST_CASE_GENERATION_WORKFLOW.md)
- Phase 1-3: Generate test case documentation (Polarion XML + Markdown)
- Phase 4: Generate actual E2E test code (.go files) in the configured E2E repository (determined by component rules)
- Use Task tool to call agents: `requirements_gathering`, `test_strategy_analysis`, `test_case_generation`, `e2e_test_generation`
- Each agent MUST follow its configuration from `config/agents/` and templates from `config/templates/`

### 2. Execute E2E Tests (Separate Workflow)
**When user asks to execute E2E tests:**
- Execute the E2E test execution workflow (independent from test generation), follow [prompts/TEST_EXECUTION_WORKFLOW.md](prompts/TEST_EXECUTION_WORKFLOW.md)
- This runs the actual E2E tests that were generated in Phase 4 of workflow #1
- Use Task tool to call agent: `test-executor` (handles all test execution phases)
- Each agent MUST follow its configuration from `config/agents/` and templates from `config/templates/`

### 3. Submit E2E Code PR
**When user asks to submit E2E code PR:**
- [In Progress]

### 4. Write Test Cases to Polarion
**When user asks to write test cases to Polarion:**
- [In Progress]
