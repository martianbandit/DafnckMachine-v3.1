---
phase: P04
step: S12
task: T06
task_id: P04-S12-T06
title: Responsive Design and Accessibility
previous_task: P04-S12-T05
next_task: P04-S12-T07
version: 3.1.0
source: Step.json
agent: "@coding-agent"
orchestrator: "@uber-orchestrator-agent"
---

# Mission Statement
Ensure the DafnckMachine v3.1 frontend is fully responsive and accessible, providing an optimal user experience across devices and for all users.

# Description
This task covers the implementation of responsive layouts, mobile-first design principles, and accessibility best practices (WCAG, ARIA, keyboard navigation). The goal is to guarantee usability and inclusivity for all users, regardless of device or ability.

# Super-Prompt
You are @coding-agent responsible for responsive design and accessibility in DafnckMachine v3.1. Your mission is to deliver a frontend that adapts to all screen sizes and meets accessibility standards, ensuring a seamless and inclusive user experience.

# MCP Tools Required
- edit_file
- file_search
- list_dir
- run_terminal_cmd
- mcp_taskmaster-ai_get_task
- mcp_taskmaster-ai_set_task_status

# Result We Want
- Responsive design implemented with mobile optimization, adaptive layouts, and cross-device compatibility verified and operational.
- Accessibility compliance implemented with WCAG standards, screen reader support, and inclusive design verified and operational.

# Add to Brain
- Responsive design system and breakpoints
- Accessibility features and compliance
- Adaptive layouts and device optimization

# Documentation & Templates
- [Responsive_Design_Guide.md](mdc:01_Machine/04_Documentation/Doc/Phase_4/12_Frontend_Development/Responsive_Design_Guide.md)
- [Breakpoint_Specifications.json](mdc:01_Machine/04_Documentation/Doc/Phase_4/12_Frontend_Development/Breakpoint_Specifications.json)
- [Accessibility_Implementation_Guide.md](mdc:01_Machine/04_Documentation/Doc/Phase_4/12_Frontend_Development/Accessibility_Implementation_Guide.md)
- [WCAG_Compliance_Checklist.json](mdc:01_Machine/04_Documentation/Doc/Phase_4/12_Frontend_Development/WCAG_Compliance_Checklist.json)

# Primary Responsible Agent
@coding-agent - responsive-design, accessibility

# Supporting Agents
- @design-system-agent
- @ux-researcher-agent

# Agent Selection Criteria
@coding-agent is selected for expertise in responsive and accessible UI. Supporting agents provide design system and UX guidance.

# Tasks (Summary)
- Configure responsive design system and breakpoints
- Implement adaptive layouts and component responsiveness
- Implement accessibility features and ARIA attributes
- Implement keyboard navigation and focus management
- Verify responsive and accessible behavior

# Subtasks (Detailed)
## Subtask 1: Responsive Design Implementation
- **ID**: P04-T06-S01
- **Description**: Implement responsive design system with mobile-first approach, breakpoint management, and adaptive layouts for optimal user experience across devices.
- **Agent Assignment**: @coding-agent
- **Documentation Links**:
  - [Responsive_Design_Guide.md](mdc:01_Machine/04_Documentation/Doc/Phase_4/12_Frontend_Development/Responsive_Design_Guide.md)
  - [Breakpoint_Specifications.json](mdc:01_Machine/04_Documentation/Doc/Phase_4/12_Frontend_Development/Breakpoint_Specifications.json)
- **Steps**:
    1. Configure responsive design system and breakpoints (edit_file)
    2. Implement adaptive layouts and component responsiveness (edit_file)
    3. Verify responsive behavior across devices (run_terminal_cmd)
- **Success Criteria**:
    - File Content Matches: "breakpoints"
    - File Content Matches: "mobile-first"
    - File Content Matches: "media-query"
    - File Content Matches: "responsive"
    - File Content Matches: "adaptive"
    - File Content Matches: "grid"
    - Output Contains: "Responsive design functional"
    - Unit Test Pass: responsive-design-test-suite
- **On Failure**: ESCALATE_TO_HUMAN (@design-system-agent) with logs
- **Max Retries**: 3
- **Integration Points**: Ensures optimal user experience across all devices and screen sizes for accessibility and usability.
- **Next Subtask**: P04-T06-S02 (Accessibility Compliance Implementation)

## Subtask 2: Accessibility Compliance Implementation
- **ID**: P04-T06-S02
- **Description**: Implement comprehensive accessibility features with WCAG compliance, screen reader support, and inclusive design for universal application access.
- **Agent Assignment**: @usability-heuristic-agent
- **Documentation Links**:
  - [Accessibility_Implementation_Guide.md](mdc:01_Machine/04_Documentation/Doc/Phase_4/12_Frontend_Development/Accessibility_Implementation_Guide.md)
  - [WCAG_Compliance_Checklist.json](mdc:01_Machine/04_Documentation/Doc/Phase_4/12_Frontend_Development/WCAG_Compliance_Checklist.json)
- **Steps**:
    1. Implement accessibility features and ARIA attributes (edit_file)
    2. Implement keyboard navigation and focus management (edit_file)
    3. Verify accessibility compliance and WCAG standards (run_terminal_cmd)
- **Success Criteria**:
    - File Content Matches: "aria-"
    - File Content Matches: "semantic"
    - File Content Matches: "screen-reader"
    - File Content Matches: "keyboard"
    - File Content Matches: "focus"
    - File Content Matches: "tab-order"
    - Output Contains: "Accessibility compliance verified"
    - Unit Test Pass: accessibility-test-suite
    - Output Contains: "WCAG AA compliant"
- **On Failure**: ESCALATE_TO_HUMAN (@ux-researcher-agent) with logs
- **Max Retries**: 3
- **Integration Points**: Ensures universal application access and compliance with accessibility standards for inclusive user experience.
- **Next Subtask**: P04-T07-S01 (Performance Optimization Implementation)

# Rollback Procedures
- Revert to previous working responsive or accessibility configuration
- Escalate to @design-system-agent or @ux-researcher-agent if repeated failures

# Integration Points
- Responsive and accessible design are core to user experience and compliance
- Enable mobile optimization and universal access

# Quality Gates
1. Responsive and accessible design must be implemented and tested
2. All tests must pass for responsive and accessible features
3. Compliance with WCAG standards must be verified

# Success Criteria
- Responsive and accessible design are functional, compliant, and tested

# Risk Mitigation
- Use version control for all responsive and accessibility changes
- Validate with automated and manual tests
- Escalate to supporting agents on persistent issues

# Output Artifacts
- Responsive design configuration
- Accessibility implementation code
- Test suites and results

# Next Action
Begin implementation of responsive design and accessibility with @coding-agent

# Post-Completion Action
Update @Step.json and @DNA.json to reflect SUCCEEDED status for this task and proceed to P04-S12-T07-Performance-Optimization-and-Testing.md 