---
phase: P03
step: S05
task: T05
task_id: P03-S05-T05
title: Project Initialization and Analysis Protocol
previous_task: P03-S05-T04
next_task: P03-S05-T06
version: 3.1.0
source: Step.json
agent: "@prd-architect-agent"
orchestrator: "@uber-orchestrator-agent"
---
## Output Artifacts Checklist
- [ ] 01_Machine/04_Documentation/vision/Phase_3/05_PRD_Generator/Project_Initialization_Protocol.md — Project_Initialization_Protocol.md: Project_Initialization_Protocol.md (missing)
- [ ] 01_Machine/04_Documentation/vision/Phase_3/05_PRD_Generator/Automated_Analysis_Framework.md — Automated_Analysis_Framework.md: Automated_Analysis_Framework.md (missing)

## Mission Statement
Define universal project specification framework and automated analysis capabilities for project initialization, market research, technical feasibility, and architecture generation.

## Description
Define universal project specification including platform selection matrix, technology preferences, performance requirements, integration needs, and constraints handling. Specify automated analysis including market research, technical feasibility, architecture generation, technology optimization, risk assessment, and resource estimation.

## Super-Prompt
You are @prd-architect-agent responsible for defining the project initialization and analysis protocol for DafnckMachine v3.1. Your mission is to specify a universal project specification framework and automated analysis capabilities, ensuring comprehensive requirement capture and informed autonomous decisions. Follow the DafnckMachine v3.1 PRD template structure exactly.

## MCP Tools Required
- edit_file
- file_search
- list_dir
- web_search

## Result We Want
- Universal project specification supporting any software type with comprehensive requirement capture
- Complete automated analysis with architecture generation enabling informed autonomous decisions

## Add to Brain
- **Project Specification**: Universal project specification including platform selection, technology preferences, performance requirements, integration needs, and constraints
- **Automated Analysis**: Automated market research, technical feasibility, architecture generation, technology optimization, risk assessment, and resource estimation

## Documentation & Templates
- [PRD_Template.md](mdc:01_Machine/04_Documentation/vision/Phase_3/PRD_Template.md)
- [Project_Initialization_Protocol.md](mdc:01_Machine/04_Documentation/vision/Phase_3/05_PRD_Generator/Project_Initialization_Protocol.md)
- [Universal_Specification_Framework.json](mdc:01_Machine/04_Documentation/vision/Phase_3/05_PRD_Generator/Universal_Specification_Framework.json)
- [Automated_Analysis_Framework.md](mdc:01_Machine/04_Documentation/vision/Phase_3/05_PRD_Generator/Automated_Analysis_Framework.md)
- [Architecture_Generation_Specifications.json](mdc:01_Machine/04_Documentation/vision/Phase_3/05_PRD_Generator/Architecture_Generation_Specifications.json)

## Primary Responsible Agent
@prd-architect-agent - project-initialization, universal-specification, requirements-analysis

## Supporting Agents
- @system-architect-agent: automated-analysis, architecture-generation, feasibility-assessment

## Agent Selection Criteria
The PRD Architect Agent is chosen for its expertise in project initialization, universal specification, and requirements analysis. The System Architect Agent supports automated analysis and architecture generation.

## Tasks (Summary)
- Define universal project specification framework and automated analysis capabilities for project initialization, market research, technical feasibility, and architecture generation.

## Subtasks (Detailed)
### Subtask-01: Universal Project Specification Framework
- **ID**: P03-T05-S01
- **Description**: Define universal project specification including platform selection matrix, technology preferences, performance requirements, integration needs, and constraints handling.
- **Agent Assignment**: @prd-architect-agent (project-initialization, universal-specification, requirements-analysis)
- **Documentation Links**: [Project_Initialization_Protocol.md](mdc:01_Machine/04_Documentation/vision/Phase_3/05_PRD_Generator/Project_Initialization_Protocol.md), [Universal_Specification_Framework.json](mdc:01_Machine/04_Documentation/vision/Phase_3/05_PRD_Generator/Universal_Specification_Framework.json)
- **Steps**:
    1. Define universal project specification framework and platform selection matrix (edit_file)
    2. Specify performance requirements and constraints handling mechanisms (edit_file)
- **Success Criteria**:
    - Platform selection matrix and technology preferences defined
    - Performance requirements and constraints handling specified
- **Integration Points**: Initialization protocol enables autonomous project setup and guides all subsequent development decisions
- **Next Subtask**: P03-T05-S02 (Automated Analysis & Architecture Generation)

### Subtask-02: Automated Analysis & Architecture Generation
- **ID**: P03-T05-S02
- **Description**: Specify automated analysis including market research, technical feasibility, architecture generation, technology optimization, risk assessment, and resource estimation.
- **Agent Assignment**: @system-architect-agent (automated-analysis, architecture-generation, feasibility-assessment)
- **Documentation Links**: [Automated_Analysis_Framework.md](mdc:01_Machine/04_Documentation/vision/Phase_3/05_PRD_Generator/Automated_Analysis_Framework.md), [Architecture_Generation_Specifications.json](mdc:01_Machine/04_Documentation/vision/Phase_3/05_PRD_Generator/Architecture_Generation_Specifications.json)
- **Steps**:
    1. Define automated market research and technical feasibility analysis (edit_file)
    2. Specify architecture generation and technology optimization processes (edit_file)
- **Success Criteria**:
    - Market research and technical feasibility defined
    - Architecture generation and technology optimization specified
- **Integration Points**: Automated analysis enables informed autonomous decisions and provides foundation for development planning
- **Next Subtask**: P03-T06-S01 (Autonomous Design Generation)

## Rollback Procedures
- Adjust project specification or analysis scope if requirement capture or feasibility is insufficient.

## Integration Points
- Initialization protocol enables autonomous project setup and guides all subsequent development decisions.

## Quality Gates
- Universal project specification supporting any software type with comprehensive requirement capture.
- Complete automated analysis with architecture generation enabling informed autonomous decisions.

## Success Criteria
- Universal project specification supporting any software type with comprehensive requirement capture.
- Complete automated analysis with architecture generation enabling informed autonomous decisions.

## Risk Mitigation
- Adjust project specification or analysis scope if requirement capture or feasibility is insufficient.

## Output Artifacts
- [Project_Initialization_Protocol.md](mdc:01_Machine/04_Documentation/vision/Phase_3/05_PRD_Generator/Project_Initialization_Protocol.md)
- [Automated_Analysis_Framework.md](mdc:01_Machine/04_Documentation/vision/Phase_3/05_PRD_Generator/Automated_Analysis_Framework.md)

## Next Action
Document universal project specification and automated analysis with @prd-architect-agent and @system-architect-agent

## Post-Completion Action
Upon successful completion of all subtasks within this tactical task, ensure the @Step.json and @DNA.json files are updated to reflect its SUCCEEDED status and any associated progress or outcomes. 