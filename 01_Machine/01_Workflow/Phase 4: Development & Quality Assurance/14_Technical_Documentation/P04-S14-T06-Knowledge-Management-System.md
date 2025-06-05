---
phase: P04
step: S14
task: T06
task_id: P04-S14-T06
title: Knowledge Management System
previous_task: P04-S14-T05
next_task: P04-S14-T07
version: 3.1.0
source: Step.json
agent: "@documentation-agent"
orchestrator: "@uber-orchestrator-agent"
---
## Output Artifacts Checklist
- [ ] 01_Machine/04_Documentation/vision/Phase_4/14_Technical_Documentation/Knowledge_Base_Structure.md — Knowledge_Base_Structure.md: Knowledge_Base_Structure.md (missing)
- [ ] 01_Machine/04_Documentation/vision/Phase_4/14_Technical_Documentation/KMS_Best_Practices.md — KMS_Best_Practices.md: KMS_Best_Practices.md (missing)

## Mission Statement
Establish a knowledge management system to organize, store, and provide access to all technical documentation and project knowledge for DafnckMachine v3.1.

## Description
Implement a knowledge management system (KMS) or platform to centralize documentation, guides, FAQs, and best practices. Ensure information is easily searchable, accessible, and up-to-date.

## Super-Prompt
You are @documentation-agent and @system-architect-agent responsible for the knowledge management system for DafnckMachine v3.1. Your mission is to ensure all project knowledge is organized, accessible, and maintained.

## MCP Tools Required
- edit_file
- file_search
- mcp_taskmaster-ai_get_task
- mcp_taskmaster-ai_set_task_status

## Result We Want
- Centralized, accessible, and well-organized knowledge management system

## Add to Brain
- Knowledge management system structure, access policies, and best practices

## Documentation & Templates
- [Knowledge_Base_Structure.md](mdc:01_Machine/04_Documentation/vision/Phase_4/14_Technical_Documentation/Knowledge_Base_Structure.md)
- [KMS_Best_Practices.md](mdc:01_Machine/04_Documentation/vision/Phase_4/14_Technical_Documentation/KMS_Best_Practices.md)

## Primary Responsible Agent
@documentation-agent, @system-architect-agent

## Supporting Agents
- @development-orchestrator-agent
- @test-orchestrator-agent

## Agent Selection Criteria
The Documentation Agent and System Architect Agent are chosen for their expertise in knowledge management, documentation, and system architecture.

## Tasks (Summary)
- Implement a knowledge management system or platform
- Organize and centralize all documentation and project knowledge
- Ensure information is accessible and up-to-date

## Subtasks (Detailed)
### Subtask-01: Implement Knowledge Management System
- **ID**: P04-T06-S01
- **Description**: Select and implement a knowledge management system or platform
- **Agent Assignment**: @system-architect-agent
- **Documentation Links**:
  - [Knowledge_Base_Structure.md](mdc:01_Machine/04_Documentation/vision/Phase_4/14_Technical_Documentation/Knowledge_Base_Structure.md)
- **Steps**:
    1. Evaluate and select KMS platform (edit_file)
    2. Implement and configure KMS (edit_file)
- **Success Criteria**:
    - KMS platform selected and implemented
- **Integration Points**: KMS must integrate with documentation and project tools
- **Next Subtask**: P04-T06-S02 (Organize and Centralize Knowledge)

### Subtask-02: Organize and Centralize Knowledge
- **ID**: P04-T06-S02
- **Description**: Organize, centralize, and maintain all documentation and project knowledge
- **Agent Assignment**: @documentation-agent
- **Documentation Links**:
  - [KMS_Best_Practices.md](mdc:01_Machine/04_Documentation/vision/Phase_4/14_Technical_Documentation/KMS_Best_Practices.md)
- **Steps**:
    1. Organize and migrate documentation to KMS (edit_file)
    2. Maintain and update knowledge base (edit_file)
- **Success Criteria**:
    - All documentation and knowledge centralized and maintained
- **Integration Points**: Knowledge base must be accessible to all stakeholders
- **Next Subtask**: P04-T07-S01 (Visual Documentation and Diagrams)

## Rollback Procedures
- Escalate to @documentation-lead or @system-architect-lead with logs if KMS implementation fails

## Integration Points
- KMS integrated with documentation, onboarding, and support

## Quality Gates
- KMS must be reviewed and approved by leads

## Success Criteria
- Knowledge management system is implemented, organized, and accessible

## Risk Mitigation
- Regular reviews and updates to KMS and knowledge base

## Output Artifacts
- [Knowledge_Base_Structure.md](mdc:01_Machine/04_Documentation/vision/Phase_4/14_Technical_Documentation/Knowledge_Base_Structure.md)
- [KMS_Best_Practices.md](mdc:01_Machine/04_Documentation/vision/Phase_4/14_Technical_Documentation/KMS_Best_Practices.md)

## Next Action
Implement and organize the knowledge management system with @documentation-agent and @system-architect-agent

## Post-Completion Action
Upon completion, update @Step.json and @DNA.json to reflect progress to the next task (P04-S14-T07-Visual-Documentation-and-Diagrams.md). 