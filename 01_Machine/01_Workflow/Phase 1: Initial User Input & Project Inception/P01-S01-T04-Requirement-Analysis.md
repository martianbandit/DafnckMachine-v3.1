---
phase: P01
step: S01
task: T04
task_id: P01-S01-T04
title: Requirement Analysis
previous_task: P01-S01-T03
next_task: P01-S01-T05
version: 3.1.0
source: Step.json
agent: "@elicitation-agent"
orchestrator: "@uber-orchestrator-agent"
---
## Output Artifacts Checklist
- [ ] 01_Machine/04_Documentation/vision/Phase_1/01_User_Briefing/Requirements.md — Requirements.md: Documented user and system requirements (missing)

## Mission Statement
Analyze and document functional requirements and technical constraints for the project.

## Description
This task focuses on gathering core features, user workflows, system capabilities, and integration needs, and documenting them in Requirements_Matrix.json. The resulting requirements matrix will drive system architecture and feature development.

## Super-Prompt
You are @elicitation-agent responsible for eliciting detailed core features, user workflows, system capabilities, integration needs, and performance expectations. Structure these within Requirements_Matrix.json.

## MCP Tools Required
- edit_file: Create and update requirements matrix documentation
- file_search: Locate existing requirements matrix or related documentation
- list_dir: Verify documentation structure

## Result We Want
- A complete functional requirements matrix documented with priorities and dependencies in Requirements_Matrix.json.

## Add to Brain
- Requirements Matrix: Functional requirements, technical constraints, and quality expectations

## Documentation & Templates
- [Requirements_Matrix.json](mdc:01_Machine/04_Documentation/vision/Phase_1/01_User_Briefing/Requirements_Matrix.json): Structured requirements and constraints
- [Briefing_Summary.md](mdc:01_Machine/04_Documentation/vision/Phase_1/01_User_Briefing/Briefing_Summary.md): Briefing session summary

## Primary Responsible Agent
@elicitation-agent

## Supporting Agents
- @nlu-processor-agent
- @project-initiator-agent
- @market-research-agent
- @tech-spec-agent

## Agent Selection Criteria
The @elicitation-agent is chosen for its expertise in functional analysis, requirement structuring, and document editing. Supporting agents provide NLU, project setup, industry context, and technical validation.

## Tasks (Summary)
- Gather and document core features, user workflows, system capabilities, and integration needs.

## Subtasks (Detailed)
### Subtask-01: Functional Requirements
- **ID**: P01-T04-S01
- **Description**: Gather core features, user workflows, system capabilities, and integration needs.
- **Agent Assignment**: @elicitation-agent
- **Documentation Links**:
  - [Requirements_Matrix.json](mdc:01_Machine/04_Documentation/vision/Phase_1/01_User_Briefing/Requirements_Matrix.json)
  - [Briefing_Summary.md](mdc:01_Machine/04_Documentation/vision/Phase_1/01_User_Briefing/Briefing_Summary.md)
- **Steps**:
    1. Elicit detailed core features, user workflows (e.g., user stories), system capabilities, integration needs, and performance expectations.
    2. Structure these within Requirements_Matrix.json as a list of at least 10 functional requirements, each with ID, description, priority, status, and dependencies (if applicable).
- **Success Criteria**:
    - Requirements_Matrix.json contains a structured list of at least 10 functional requirements, each with ID, description, priority, status, and dependencies.
    - Requirements_Matrix.json is a valid JSON document and adheres to the expected schema for requirements matrices.
- **Max Retries**: 2
- **On Failure**: ESCALATE_TO_HUMAN (@requirements-lead) with logs and current draft.
- **Integration Points**: Functional requirements will directly drive the system architecture and detailed design phases.
- **Next Subtask**: P01-S01-T06-Technical-Constraints.md

## Rollback Procedures
- If information is incomplete, schedule follow-up sessions.
- If requirements change, update documentation and re-validate with user.

## Integration Points
- Functional requirements drive system architecture and detailed design phases.

## Quality Gates
- Completeness Check: All required requirement fields are filled.
- Clarity Validation: Requirements are clearly articulated and unambiguous.
- Documentation Standards: Requirements_Matrix.json is valid and structured.

## Success Criteria
- Comprehensive functional requirements matrix with priorities established.

## Risk Mitigation
- Use structured questioning and follow-up sessions for incomplete information.
- Employ clarification techniques for unclear responses.

## Output Artifacts
- [Requirements_Matrix.json](mdc:01_Machine/04_Documentation/vision/Phase_1/01_User_Briefing/Requirements_Matrix.json): Structured functional and technical requirements

## Next Action
Initiate requirements analysis and document findings.

## Post-Completion Action
Update Step.json and DNA.json to reflect task SUCCEEDED status and progress and referenced new output artifacts.
Upon completion, proceed to P01-S01-T06-Technical-Constraints.md and update Step.json and DNA.json to reflect progress. 