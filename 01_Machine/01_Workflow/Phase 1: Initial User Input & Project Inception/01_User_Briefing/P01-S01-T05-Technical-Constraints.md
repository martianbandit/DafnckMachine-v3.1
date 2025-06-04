---
phase: P01
step: S01
task: T05
task_id: P01-S01-T05
title: Technical Constraints
previous_task: P01-S01-T04
next_task: P02-S01-T01
version: 3.1.0
source: Step.json
agent: "@tech-spec-agent"
orchestrator: "@uber-orchestrator-agent"
---

## Mission Statement
Identify platform preferences, technology stack, security, scalability, and integration limitations, and document technical constraints for the project.

## Description
This task focuses on identifying and documenting all technical constraints, including platform and technology preferences, security and scalability requirements, and integration limitations. A preliminary feasibility assessment is also performed.

## Super-Prompt
You are @tech-spec-agent responsible for eliciting and documenting platform preferences, technology stack requirements, security requirements, scalability needs, and integration limitations. Document these in Constraints_Matrix.json and perform a preliminary feasibility assessment.

## MCP Tools Required
- edit_file: Create and update constraints matrix documentation
- file_search: Locate existing constraints matrix or related documentation
- list_dir: Verify documentation structure

## Result We Want
- Technical constraints documented with a preliminary feasibility assessment in Constraints_Matrix.json.

## Add to Brain
- Constraints Matrix: Platform preferences, technology stack, security, scalability, and integration limitations

## Documentation & Templates
- [Constraints_Matrix.json](mdc:01_Machine/04_Documentation/Doc/Phase_1/01_User_Briefing/Constraints_Matrix.json): Comprehensive project constraints and limitations
- [Requirements_Matrix.json](mdc:01_Machine/04_Documentation/Doc/Phase_1/01_User_Briefing/Requirements_Matrix.json): Structured requirements and constraints

## Primary Responsible Agent
@tech-spec-agent

## Supporting Agents
- @nlu-processor-agent
- @elicitation-agent
- @project-initiator-agent
- @market-research-agent

## Agent Selection Criteria
The @tech-spec-agent is chosen for its expertise in constraint analysis, technical validation, and document editing. Supporting agents provide NLU, requirement gathering, project setup, and industry context.

## Tasks (Summary)
- Identify and document all technical constraints, including platform and technology preferences, security and scalability requirements, and integration limitations.

## Subtasks (Detailed)
### Subtask-01: Technical Constraints Assessment
- **ID**: P01-T05-S01
- **Description**: Identify and document all technical constraints, including platform and technology preferences, security and scalability requirements, and integration limitations.
- **Agent Assignment**: @tech-spec-agent
- **Documentation Links**:
  - [Constraints_Matrix.json](mdc:01_Machine/04_Documentation/Doc/Phase_1/01_User_Briefing/Constraints_Matrix.json)
  - [Requirements_Matrix.json](mdc:01_Machine/04_Documentation/Doc/Phase_1/01_User_Briefing/Requirements_Matrix.json)
- **Steps**:
    1. Elicit from the user and technical documentation any platform preferences, technology stack requirements, security requirements (e.g., compliance), scalability needs, and integration limitations.
    2. Document these in Constraints_Matrix.json as at least 5 distinct technical constraints, each with ID, type, description, severity, and feasibility_assessment.
    3. Perform a preliminary feasibility assessment for each constraint.
- **Success Criteria**:
    - Constraints_Matrix.json contains at least 5 distinct technical constraints, each with ID, type, description, severity, and feasibility_assessment.
    - Constraints_Matrix.json is a valid JSON document and adheres to the expected schema.
- **Max Retries**: 2
- **On Failure**: ESCALATE_TO_HUMAN (@tech-lead) with logs and identified constraints.
- **Integration Points**: Technical constraints will critically inform architecture and technology stack decisions.
- **Next Subtask**: P01-S01-T07-Context-and-Market-Analysis.md

## Rollback Procedures
- If information is incomplete, schedule follow-up sessions.
- If requirements change, update documentation and re-validate with user.

## Integration Points
- Technical constraints inform architecture and technology stack decisions.

## Quality Gates
- Completeness Check: All required constraint fields are filled.
- Clarity Validation: Constraints are clearly articulated and unambiguous.
- Documentation Standards: Constraints_Matrix.json is valid and structured.

## Success Criteria
- Technical constraints and feasibility assessment completed and documented.

## Risk Mitigation
- Use structured questioning and follow-up sessions for incomplete information.
- Employ clarification techniques for unclear responses.

## Output Artifacts
- [Constraints_Matrix.json](mdc:01_Machine/04_Documentation/vision/Phase_1/01_User_Briefing/Constraints_Matrix.json): Comprehensive project constraints and limitations

## Next Action
Initiate technical constraints assessment and document findings.

## Post-Completion Action
Update Step.json and DNA.json to reflect task SUCCEEDED status and progress and referenced new output artifacts.
Upon completion, proceed to P02-S01-T01-Context-and-Market-Analysis.md and update Step.json and DNA.json to reflect progress. 