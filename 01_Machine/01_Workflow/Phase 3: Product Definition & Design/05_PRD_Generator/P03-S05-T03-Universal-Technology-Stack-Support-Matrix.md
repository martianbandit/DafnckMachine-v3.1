---
phase: P03
step: S05
task: T03
task_id: P03-S05-T03
title: Universal Technology Stack Support Matrix
previous_task: P03-S05-T02
next_task: P03-S05-T04
version: 3.1.0
source: Step.json
agent: "@technology-advisor-agent"
orchestrator: "@uber-orchestrator-agent"
---
## Output Artifacts Checklist
- [ ] 01_Machine/04_Documentation/vision/Phase_3/05_PRD_Generator/Technology_Stack_Matrix.md — Technology_Stack_Matrix.md: Technology_Stack_Matrix.md (missing)
- [ ] 01_Machine/04_Documentation/vision/Phase_3/05_PRD_Generator/Design_System_Integration.md — Design_System_Integration.md: Design_System_Integration.md (missing)

## Mission Statement
Define universal technology support covering web applications, mobile development, desktop applications, system programming, and all major development platforms with cross-platform consistency.

## Description
Define universal technology support for web applications, mobile development, desktop applications, system programming, game development, data science, blockchain, and enterprise systems.

## Super-Prompt
You are @technology-advisor-agent responsible for defining the universal technology stack support matrix for DafnckMachine v3.1. Your mission is to ensure support for all major development platforms, frameworks, and languages, enabling cross-platform consistency and platform-specific adaptations. Follow the DafnckMachine v3.1 PRD template structure exactly.

## MCP Tools Required
- edit_file
- file_search
- list_dir
- web_search

## Result We Want
- Comprehensive technology matrix supporting any project type with platform-specific implementations
- Universal support for web, mobile, desktop, system, game, data science, blockchain, and enterprise

## Add to Brain
- **Technology Matrix**: Universal support for any technology stack

## Documentation & Templates
- [PRD_Template.md](mdc:01_Machine/04_Documentation/vision/Phase_3/PRD_Template.md)
- [Technology_Stack_Matrix.md](mdc:01_Machine/04_Documentation/vision/Phase_3/05_PRD_Generator/Technology_Stack_Matrix.md)
- [Platform_Support_Specifications.json](mdc:01_Machine/04_Documentation/vision/Phase_3/05_PRD_Generator/Platform_Support_Specifications.json)
- [Design_System_Integration.md](mdc:01_Machine/04_Documentation/vision/Phase_3/05_PRD_Generator/Design_System_Integration.md)
- [UI_Framework_Matrix.json](mdc:01_Machine/04_Documentation/vision/Phase_3/05_PRD_Generator/UI_Framework_Matrix.json)

## Primary Responsible Agent
@technology-advisor-agent - universal-support, cross-platform, technology-integration

## Supporting Agents
- @technology-advisor-agent: design-systems, ui-frameworks, accessibility-standards

## Agent Selection Criteria
The Technology Advisor Agent is chosen for its expertise in universal technology support, cross-platform integration, and design system standards.

## Tasks (Summary)
- Define universal technology support covering all major development platforms with cross-platform consistency.

## Subtasks (Detailed)
### Subtask-01: Cross-Platform Technology Integration
- **ID**: P03-T03-S01
- **Description**: Define universal technology support for web applications, mobile development, desktop applications, system programming, game development, data science, blockchain, and enterprise systems.
- **Agent Assignment**: @technology-advisor-agent (universal-support, cross-platform, technology-integration)
- **Documentation Links**: [Technology_Stack_Matrix.md](mdc:01_Machine/04_Documentation/vision/Phase_3/05_PRD_Generator/Technology_Stack_Matrix.md), [Platform_Support_Specifications.json](mdc:01_Machine/04_Documentation/vision/Phase_3/05_PRD_Generator/Platform_Support_Specifications.json)
- **Steps**:
    1. Research and document comprehensive technology stack support matrix (web_search)
    2. Create technology matrix covering all major development platforms (edit_file)
- **Success Criteria**:
    - Technology research completed
    - Technology matrix covers web, mobile, desktop, system, game, data science
    - File: Technology_Stack_Matrix.md
- **Integration Points**: Universal support enables autonomous development for any technology stack or platform
- **Next Subtask**: P03-T03-S02 (Design System & UI Framework Integration)

### Subtask-02: Design System & UI Framework Integration
- **ID**: P03-T03-S02
- **Description**: Specify universal design system integration with Shadcn/ui + Tailwind CSS foundation, platform-specific adaptations, cross-platform consistency, and accessibility standards.
- **Agent Assignment**: @technology-advisor-agent (design-systems, ui-frameworks, accessibility-standards)
- **Documentation Links**: [Design_System_Integration.md](mdc:01_Machine/04_Documentation/vision/Phase_3/05_PRD_Generator/Design_System_Integration.md), [UI_Framework_Matrix.json](mdc:01_Machine/04_Documentation/vision/Phase_3/05_PRD_Generator/UI_Framework_Matrix.json)
- **Steps**:
    1. Specify Shadcn/ui + Tailwind CSS foundation integration (edit_file)
    2. Define accessibility standards and cross-platform consistency requirements (edit_file)
- **Success Criteria**:
    - Shadcn/ui + Tailwind CSS foundation and platform-specific adaptations specified
    - Accessibility standards and cross-platform consistency defined
- **Integration Points**: Design system ensures consistent user experience across platforms and enables automated UI generation
- **Next Subtask**: P03-T04-S01 (Language-Specific Development Agents)

## Rollback Procedures
- Refine universal support matrix or focus on core platforms if technology limitations are found.

## Integration Points
- Universal support enables autonomous development for any technology stack or platform.

## Quality Gates
- Universal Support: Technology matrix supports any software development project.
- Design System: Ensures consistent user experience across platforms.

## Success Criteria
- Universal technology stack support matrix enabling any project type.
- Universal design system with platform-specific implementations ensuring consistent user experience.

## Risk Mitigation
- Refine universal support matrix or focus on core platforms if technology limitations are found.

## Output Artifacts
- [Technology_Stack_Matrix.md](mdc:01_Machine/04_Documentation/vision/Phase_3/05_PRD_Generator/Technology_Stack_Matrix.md)
- [Design_System_Integration.md](mdc:01_Machine/04_Documentation/vision/Phase_3/05_PRD_Generator/Design_System_Integration.md)

## Next Action
Document universal technology stack support and design system integration with @technology-advisor-agent

## Post-Completion Action
Upon successful completion of all subtasks within this tactical task, ensure the @Step.json and @DNA.json files are updated to reflect its SUCCEEDED status and any associated progress or outcomes. 