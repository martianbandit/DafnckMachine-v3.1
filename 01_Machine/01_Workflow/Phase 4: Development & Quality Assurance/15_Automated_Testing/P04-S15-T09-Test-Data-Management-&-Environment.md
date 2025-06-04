---
phase: P04
step: S15
task: T09
task_id: P04-S15-T09
title: Test Data Management & Environment
previous_task: P04-S15-T08
next_task: P04-S15-T10
version: 3.1.0
source: Step.json
agent: "@test-orchestrator-agent"
orchestrator: "@uber-orchestrator-agent"
---

# Mission Statement
Implement comprehensive test data management with test data generation, data seeding, data cleanup, and privacy compliance for reliable testing environments in DafnckMachine v3.1.

# Description
This task covers the development and execution of test data management processes, including test data generation, seeding, cleanup, and privacy compliance to ensure reliable and consistent testing environments.

# Super-Prompt
You are @test-orchestrator-agent responsible for implementing test data management and environment isolation for DafnckMachine v3.1. Your mission is to develop and execute test data generation, seeding, cleanup, and privacy compliance to ensure reliable and consistent testing environments.

# MCP Tools Required
- edit_file
- run_terminal_cmd

# Result We Want
- Comprehensive test data management and environment isolation
- Automated data generation, seeding, and cleanup implemented

# Add to Brain
- Test data management and environment isolation

# Documentation & Templates
- [Test_Data_Management.md](mdc:01_Machine/04_Documentation/Doc/Phase_4/15_Automated_Testing/Test_Data_Management.md)
- [Data_Generation_Framework.json](mdc:01_Machine/04_Documentation/Doc/Phase_4/15_Automated_Testing/Data_Generation_Framework.json)
- [Test_Environment_Management.md](mdc:01_Machine/04_Documentation/Doc/Phase_4/15_Automated_Testing/Test_Environment_Management.md)
- [Environment_Isolation_Setup.json](mdc:01_Machine/04_Documentation/Doc/Phase_4/15_Automated_Testing/Environment_Isolation_Setup.json)

# Primary Responsible Agent
@test-orchestrator-agent

# Supporting Agents
- @functional-tester-agent

# Agent Selection Criteria
The @test-orchestrator-agent is selected for its expertise in test data management and environment isolation. The @functional-tester-agent supports with test development and validation.

# Tasks (Summary)
- Develop and execute test data management and environment isolation
- Implement automated data generation, seeding, and cleanup

# Subtasks (Detailed)
## Subtask 1: Test Data Generation & Management
- **ID**: P04-T09-S15
- **Description**: Develop test data management system with automated generation and privacy compliance.
- **Agent**: @test-orchestrator-agent
- **Documentation**: [Test_Data_Management.md](mdc:01_Machine/04_Documentation/Doc/Phase_4/15_Automated_Testing/Test_Data_Management.md), [Data_Generation_Framework.json](mdc:01_Machine/04_Documentation/Doc/Phase_4/15_Automated_Testing/Data_Generation_Framework.json)
- **Steps**:
    1. Implement test data management with automated generation and privacy compliance (edit_file)
    2. Setup data seeding and cleanup automation (run_terminal_cmd)
- **Success Criteria**:
    - Test data management documented and covers data generation, seeding, cleanup, and privacy compliance
    - Test data management configured successfully

## Subtask 2: Environment Isolation & Management
- **ID**: P04-T09-S16
- **Description**: Develop environment management system with isolation, provisioning, configuration management, and cleanup automation.
- **Agent**: @test-orchestrator-agent
- **Documentation**: [Test_Environment_Management.md](mdc:01_Machine/04_Documentation/Doc/Phase_4/15_Automated_Testing/Test_Environment_Management.md), [Environment_Isolation_Setup.json](mdc:01_Machine/04_Documentation/Doc/Phase_4/15_Automated_Testing/Environment_Isolation_Setup.json)
- **Steps**:
    1. Implement environment management with isolation and automated provisioning (edit_file)
    2. Configure environment cleanup and management automation (run_terminal_cmd)
- **Success Criteria**:
    - Environment management documented and covers isolation, provisioning, configuration, and cleanup
    - Environment management configured successfully

# Rollback Procedures
- Debug and fix failing test data or environment management
- Revert to previous implementation if validation fails

# Integration Points
- Test data management ensures reliable and consistent testing environments

# Quality Gates
- All test data and environment management integrations pass

# Success Criteria
- Application meets test data and environment management requirements

# Risk Mitigation
- Escalate to @test-data-lead or @environment-management-lead if tests fail after 3 attempts

# Output Artifacts
- [Test_Data_Management.md](mdc:01_Machine/04_Documentation/Doc/Phase_4/15_Automated_Testing/Test_Data_Management.md)
- [Data_Generation_Framework.json](mdc:01_Machine/04_Documentation/Doc/Phase_4/15_Automated_Testing/Data_Generation_Framework.json)
- [Test_Environment_Management.md](mdc:01_Machine/04_Documentation/Doc/Phase_4/15_Automated_Testing/Test_Environment_Management.md)
- [Environment_Isolation_Setup.json](mdc:01_Machine/04_Documentation/Doc/Phase_4/15_Automated_Testing/Environment_Isolation_Setup.json)

# Next Action
Develop and execute test data management and environment isolation with @test-orchestrator-agent

# Post-Completion Action
Proceed to P04-S15-T10-Test-Reporting-&-Analytics.md 