---
phase: P01
step: S01
task: T03
task_id: P01-S01-T03
title: Success Criteria Definition
previous_task: P01-S01-T02
next_task: P01-S01-T04
version: 3.1.0
source: Step.json
agent: "@elicitation-agent"
orchestrator: "@uber-orchestrator-agent"
---
## Output Artifacts Checklist
- [ ] 01_Machine/04_Documentation/vision/Phase_1/01_User_Briefing/Success_Criteria.md — Success_Criteria.md: Documented success criteria and measurable goals (missing)

## Mission Statement
Define measurable outcomes, KPIs, acceptance criteria, and quality standards for project success.

## Description
This task focuses on working with the user to establish clear and quantifiable success metrics, KPIs, acceptance criteria for core features, and overall quality standards. These will be documented in Success_Metrics.json and guide project evaluation.

## Super-Prompt
You are @elicitation-agent responsible for collaborating with the user to define measurable outcomes, key performance indicators (KPIs), acceptance criteria for key features, and quality standards. Record these in Success_Metrics.json.

## MCP Tools Required
- edit_file: Create and update success metrics documentation
- file_search: Locate existing success metrics or related documentation
- list_dir: Verify documentation structure

## Result We Want
- Quantifiable success metrics and acceptance criteria fully established and documented in Success_Metrics.json.

## Add to Brain
- Success Metrics: Measurable outcomes, KPIs, acceptance criteria, and quality standards

## Documentation & Templates
- [Success_Metrics.json](mdc:01_Machine/04_Documentation/vision/Phase_1/01_User_Briefing/Success_Metrics.json): Measurable success criteria and KPIs
- [Project_Vision_Statement.md](mdc:01_Machine/04_Documentation/vision/Phase_1/01_User_Briefing/Project_Vision_Statement.md): Core project concept and objectives

## Primary Responsible Agent
@elicitation-agent

## Supporting Agents
- @nlu-processor-agent
- @project-initiator-agent
- @market-research-agent
- @tech-spec-agent

## Agent Selection Criteria
The @elicitation-agent is chosen for its expertise in success metrics, criteria definition, and document editing. Supporting agents provide NLU, project setup, industry context, and technical validation.

## Tasks (Summary)
- Define and document measurable outcomes, KPIs, acceptance criteria, and quality standards for project success.

## Subtasks (Detailed)
### Subtask-01: Success Criteria Definition
- **ID**: P01-T03-S01
- **Description**: Define measurable outcomes, KPIs, acceptance criteria, and quality standards.
- **Agent Assignment**: @elicitation-agent
- **Documentation Links**:
  - [Success_Metrics.json](mdc:01_Machine/04_Documentation/vision/Phase_1/01_User_Briefing/Success_Metrics.json)
  - [Project_Vision_Statement.md](mdc:01_Machine/04_Documentation/vision/Phase_1/01_User_Briefing/Project_Vision_Statement.md)
- **Steps**:
    1. Work with the user to establish clear and quantifiable success metrics, KPIs, acceptance criteria for core features, and quality standards.
    2. Record these in Success_Metrics.json as a JSON array or object with at least 3 distinct, measurable success metrics/KPIs, and at least 5 acceptance criteria for core features.
- **Success Criteria**:
    - Success_Metrics.json contains at least 3 distinct, measurable success metrics/KPIs, and at least 5 acceptance criteria for core features.
    - Each metric is SMART (Specific, Measurable, Achievable, Relevant, Time-bound).
- **Max Retries**: 2
- **On Failure**: ESCALATE_TO_HUMAN (@project-lead) with logs and draft metrics.
- **Integration Points**: The defined success metrics will directly inform testing strategies, validation processes, and overall project evaluation.
- **Next Subtask**: P01-S01-T04-Requirement-Analysis.md

## Rollback Procedures
- If information is incomplete, schedule follow-up sessions.
- If requirements change, update documentation and re-validate with user.

## Integration Points
- Success metrics inform testing strategies, validation processes, and project evaluation.

## Quality Gates
- Completeness Check: All required success metric fields are filled.
- Clarity Validation: Metrics are clearly articulated and unambiguous.
- Documentation Standards: Success_Metrics.json is valid and structured.

## Success Criteria
- Quantifiable success metrics and acceptance criteria are fully established and documented.

## Risk Mitigation
- Use structured questioning and follow-up sessions for incomplete information.
- Employ clarification techniques for unclear responses.

## Output Artifacts
- [Success_Metrics.json](mdc:01_Machine/04_Documentation/vision/Phase_1/01_User_Briefing/Success_Metrics.json): Measurable success criteria and KPIs

## Next Action
Initiate success criteria definition and document findings.

## Post-Completion Action
Update Step.json and DNA.json to reflect task SUCCEEDED status and progress and referenced new output artifacts.
Upon completion, proceed to P01-S01-T04-Requirement-Analysis.md and update Step.json and DNA.json to reflect progress. 