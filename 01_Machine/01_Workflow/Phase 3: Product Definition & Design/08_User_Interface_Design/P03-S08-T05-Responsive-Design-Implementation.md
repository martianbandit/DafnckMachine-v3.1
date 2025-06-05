---
phase: P03
step: S08
task: T05
task_id: P03-S08-T05
title: Responsive Design Implementation
previous_task: P03-S08-T04
next_task: P03-S08-T06
version: 3.1.0
source: Step.json
agent: "@ui-designer-agent"
orchestrator: "@uber-orchestrator-agent"
---
## Output Artifacts Checklist
- [x] 01_Machine/04_Documentation/vision/Phase_3/08_User_Interface_Design/Responsive_Interface_Specifications.md — Responsive Interface Specifications
- [x] 01_Machine/04_Documentation/vision/Phase_3/08_User_Interface_Design/Breakpoint_Guidelines.json — Breakpoint Guidelines
- [x] 01_Machine/04_Documentation/vision/Phase_3/08_User_Interface_Design/Multi_Device_Specifications.md — Multi-Device Interface Specifications
- [x] 01_Machine/04_Documentation/vision/Phase_3/08_User_Interface_Design/Device_Adaptation_Guidelines.json — Device Adaptation Guidelines

# Mission Statement
Define and document the responsive design strategy and create designs for multi-device interfaces for DafnckMachine v3.1.

# Description
This task covers the design of a comprehensive breakpoint strategy, layout adaptations, content prioritization, touch optimization, and the creation of multi-device interface specifications for mobile, tablet, and desktop.

# Super-Prompt
You are @ui-designer-agent. Your mission is to define a robust responsive design strategy and create detailed specifications for multi-device interfaces, ensuring optimal user experience across all platforms.

# MCP Tools Required
- edit_file

# Result We Want
- Comprehensive responsive design strategy documented
- Clear breakpoint definitions, layout adaptation rules, and touch optimization
- Multi-device interface specifications

# Add to Brain
- Responsive Framework
- Breakpoint Guidelines
- Device Adaptation Guidelines

# Documentation & Templates
- [Responsive Interface Specifications](mdc:01_Machine/04_Documentation/vision/Phase_3/08_User_Interface_Design/Responsive_Interface_Specifications.md)
- [Breakpoint Guidelines](mdc:01_Machine/04_Documentation/vision/Phase_3/08_User_Interface_Design/Breakpoint_Guidelines.json)
- [Multi-Device Interface Specifications](mdc:01_Machine/04_Documentation/vision/Phase_3/08_User_Interface_Design/Multi_Device_Specifications.md)
- [Device Adaptation Guidelines](mdc:01_Machine/04_Documentation/vision/Phase_3/08_User_Interface_Design/Device_Adaptation_Guidelines.json)

# Primary Responsible Agent
@ui-designer-agent

# Supporting Agents
- @design-system-agent

# Agent Selection Criteria
The UI Designer Agent is chosen for its expertise in responsive design, breakpoint planning, and multi-device adaptation. The Design System Agent supports consistency.

# Tasks (Summary)
1. Design breakpoint strategy (device targets, breakpoints, layout adaptations, content prioritization, touch optimization)
2. Design multi-device interface adaptations (mobile, tablet, desktop, touch/gesture support)

# Subtasks (Detailed)
## Subtask 1: Breakpoint Strategy Design
- **ID**: P03-T05-S01
- **Description**: Design a comprehensive breakpoint strategy, including device targets, breakpoint definitions, layout adaptations, content prioritization, and touch optimization.
- **Agent**: @ui-designer-agent
- **Documentation Links**:
  - [Responsive Interface Specifications](mdc:01_Machine/04_Documentation/vision/Phase_3/08_User_Interface_Design/Responsive_Interface_Specifications.md)
  - [Breakpoint Guidelines](mdc:01_Machine/04_Documentation/vision/Phase_3/08_User_Interface_Design/Breakpoint_Guidelines.json)
- **Steps**:
  1. Define target devices (mobile, tablet, desktop) and establish specific breakpoints. (edit_file)
  2. Document layout adaptation rules, content prioritization strategies, and touch optimization for each breakpoint. (edit_file)
- **Success Criteria**:
  - File Exists: Breakpoint_Guidelines.json
  - File Content Matches: 'breakpoints': [{ 'name': 'mobile', 'maxWidth': ... }]
  - File Exists: Responsive_Interface_Specifications.md
  - File Content Matches: 'layoutAdaptations': { 'mobile': ... }
  - File Content Matches: 'touchOptimization': { 'defined': true }

## Subtask 2: Multi-Device Interface Design
- **ID**: P03-T05-S02
- **Description**: Design specific interface adaptations for multiple devices, including mobile layouts, tablet adaptations, desktop optimizations, and considerations for touch interactions and gesture support.
- **Agent**: @ui-designer-agent
- **Documentation Links**:
  - [Multi-Device Interface Specifications](mdc:01_Machine/04_Documentation/vision/Phase_3/08_User_Interface_Design/Multi_Device_Specifications.md)
  - [Device Adaptation Guidelines](mdc:01_Machine/04_Documentation/vision/Phase_3/08_User_Interface_Design/Device_Adaptation_Guidelines.json)
- **Steps**:
  1. Create mockups or detailed specifications for mobile layouts, tablet adaptations, and desktop optimizations. (edit_file)
  2. Specify touch interactions and gesture support for relevant devices. (edit_file)
- **Success Criteria**:
  - File Exists: Multi_Device_Specifications.md
  - File Content Matches: 'mobileLayouts': { 'defined_for_key_screens': true }
  - File Content Matches: 'touchInteractions': { 'defined': true }
  - File Content Matches: 'gestureSupport': { 'defined': true }

# Rollback Procedures
- Refine responsive strategy and device adaptations based on feedback

# Integration Points
- Ensures optimal and consistent user experience across all targeted device types

# Quality Gates
- Visual Quality
- Design Consistency
- Implementation Feasibility

# Success Criteria
- Comprehensive responsive design strategy documented
- Clear breakpoint definitions, layout adaptation rules, and touch optimization
- Multi-device interface specifications

# Risk Mitigation
- Rigorous quality assurance process with multiple validation checkpoints
- Proactive feedback and iteration

# Output Artifacts
- [Responsive_Interface_Specifications.md](mdc:01_Machine/04_Documentation/vision/Phase_3/08_User_Interface_Design/Responsive_Interface_Specifications.md)
- [Breakpoint_Guidelines.json](mdc:01_Machine/04_Documentation/vision/Phase_3/08_User_Interface_Design/Breakpoint_Guidelines.json)
- [Multi_Device_Specifications.md](mdc:01_Machine/04_Documentation/vision/Phase_3/08_User_Interface_Design/Multi_Device_Specifications.md)
- [Device_Adaptation_Guidelines.json](mdc:01_Machine/04_Documentation/vision/Phase_3/08_User_Interface_Design/Device_Adaptation_Guidelines.json)

# Next Action
Initiate responsive design strategy and multi-device interface design with @ui-designer-agent

# Post-Completion Action
Update @Step.json and @DNA.json to reflect SUCCEEDED status and progress for this task. 