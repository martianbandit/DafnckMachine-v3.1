---
phase: P03
step: S08
task: T03
task_id: P03-S08-T03
title: Component Library Development
previous_task: P03-S08-T02
next_task: P03-S08-T04
version: 3.1.0
source: Step.json
agent: "@ui-designer-agent"
orchestrator: "@uber-orchestrator-agent"
---
## Output Artifacts Checklist
- [ ] 01_Machine/04_Documentation/vision/Phase_3/08_User_Interface_Design/Basic_Component_Library.md — Basic Component Library (missing)
- [ ] 01_Machine/04_Documentation/vision/Phase_3/08_User_Interface_Design/Component_Specifications.json — Component Specifications (missing)
- [ ] 01_Machine/04_Documentation/vision/Phase_3/08_User_Interface_Design/Complex_Component_Library.md — Complex Component Library (missing)
- [ ] 01_Machine/04_Documentation/vision/Phase_3/08_User_Interface_Design/Advanced_Component_Specs.json — Advanced Component Specifications (missing)

# Mission Statement
Develop a library of reusable UI components, covering both basic and complex elements for DafnckMachine v3.1.

# Description
This task covers the design and documentation of fundamental and advanced UI components, ensuring consistency with the design system and integration of Shadcn UI where appropriate.

# Super-Prompt
You are @ui-designer-agent. Your mission is to design and document a comprehensive library of reusable UI components, including both basic and complex elements, with detailed specifications and state variations.

# MCP Tools Required
- edit_file
- mcp_ui_Docs_search_ui_documentation

# Result We Want
- Complete basic and complex component library documented
- Detailed specifications, state variations, and integration of Shadcn UI components

# Add to Brain
- Component Library
- Component Specifications

# Documentation & Templates
- [Basic Component Library](mdc:01_Machine/04_Documentation/vision/Phase_3/08_User_Interface_Design/Basic_Component_Library.md)
- [Component Specifications](mdc:01_Machine/04_Documentation/vision/Phase_3/08_User_Interface_Design/Component_Specifications.json)
- [Complex Component Library](mdc:01_Machine/04_Documentation/vision/Phase_3/08_User_Interface_Design/Complex_Component_Library.md)
- [Advanced Component Specifications](mdc:01_Machine/04_Documentation/vision/Phase_3/08_User_Interface_Design/Advanced_Component_Specs.json)

# Primary Responsible Agent
@ui-designer-agent

# Supporting Agents
- @design-system-agent

# Agent Selection Criteria
The UI Designer Agent is chosen for its expertise in component design, basic and advanced elements, and Shadcn UI integration. The Design System Agent supports component consistency.

# Tasks (Summary)
1. Design basic UI components (buttons, inputs, labels, icons, cards, lists, navigation, forms)
2. Design complex UI components (modals, tables, charts, dashboards, multi-step forms, data visualization)

# Subtasks (Detailed)
## Subtask 1: Basic Component Design
- **ID**: P03-T03-S01
- **Description**: Design fundamental UI components such as buttons, inputs, labels, icons, cards, lists, navigation elements, and form components.
- **Agent**: @ui-designer-agent
- **Documentation Links**:
  - [Basic Component Library](mdc:01_Machine/04_Documentation/vision/Phase_3/08_User_Interface_Design/Basic_Component_Library.md)
  - [Component Specifications](mdc:01_Machine/04_Documentation/vision/Phase_3/08_User_Interface_Design/Component_Specifications.json)
- **Steps**:
  1. Design buttons, inputs, labels, and icons, specifying states (default, hover, active, disabled). (mcp_ui_Docs_search_ui_documentation, edit_file)
  2. Design cards, lists, navigation elements, and form components. (mcp_ui_Docs_search_ui_documentation, edit_file)
- **Success Criteria**:
  - File Exists: Component_Specifications.json
  - File Content Matches: 'buttons': { 'specsDefined': true, 'states': [ 'default', 'hover', 'active', 'disabled' ] }
  - File Exists: Basic_Component_Library.md
  - File Content Matches: 'navigationElements': { 'documented': true }

## Subtask 2: Complex Component Design
- **ID**: P03-T03-S02
- **Description**: Design more complex UI components such as modals, tables, charts, dashboards, multi-step forms, and data visualization elements.
- **Agent**: @ui-designer-agent
- **Documentation Links**:
  - [Complex Component Library](mdc:01_Machine/04_Documentation/vision/Phase_3/08_User_Interface_Design/Complex_Component_Library.md)
  - [Advanced Component Specifications](mdc:01_Machine/04_Documentation/vision/Phase_3/08_User_Interface_Design/Advanced_Component_Specs.json)
- **Steps**:
  1. Design modals, tables, and charts, specifying interactions and data display. (edit_file)
  2. Design dashboards, multi-step forms, and data visualization components. (edit_file)
- **Success Criteria**:
  - File Exists: Advanced_Component_Specs.json
  - File Content Matches: 'modals': { 'specsDefined': true }
  - File Exists: Complex_Component_Library.md
  - File Content Matches: 'dashboards': { 'documented': true }

# Rollback Procedures
- Refine component specifications and documentation based on feedback

# Integration Points
- These basic and complex components will form the foundation for constructing all interface sections and views

# Quality Gates
- Visual Quality
- Design Consistency
- Implementation Feasibility

# Success Criteria
- Complete basic and complex component library documented
- Detailed specifications, state variations, and integration of Shadcn UI components

# Risk Mitigation
- Rigorous quality assurance process with multiple validation checkpoints
- Proactive feedback and iteration

# Output Artifacts
- [Basic_Component_Library.md](mdc:01_Machine/04_Documentation/vision/Phase_3/08_User_Interface_Design/Basic_Component_Library.md)
- [Component_Specifications.json](mdc:01_Machine/04_Documentation/vision/Phase_3/08_User_Interface_Design/Component_Specifications.json)
- [Complex_Component_Library.md](mdc:01_Machine/04_Documentation/vision/Phase_3/08_User_Interface_Design/Complex_Component_Library.md)
- [Advanced_Component_Specs.json](mdc:01_Machine/04_Documentation/vision/Phase_3/08_User_Interface_Design/Advanced_Component_Specs.json)

# Next Action
Initiate basic and complex component design with @ui-designer-agent

# Post-Completion Action
Update @Step.json and @DNA.json to reflect SUCCEEDED status and progress for this task. 