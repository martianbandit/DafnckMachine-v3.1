---
phase: P01
step: S01
task: T02
task_id: P01-S01-T02
title: Project Vision Elicitation
previous_task: P01-S01-T01
next_task: P01-S01-T03
version: 3.1.0
source: Step.json
agent: "@elicitation-agent"
orchestrator: "@uber-orchestrator-agent"
---
## Output Artifacts Checklist
- [ ] 01_Machine/04_Documentation/vision/Phase_1/01_User_Briefing/Project_Vision.md — Project_Vision.md: Documented project vision and goals (missing)

## Mission Statement
Elicit the core concept and define measurable success criteria for the project, including main objectives, target audience, key features, and unique value proposition.

## Description
This task focuses on discovering the main objectives, target audience, key features, and unique value proposition of the project. The resulting vision statement will guide all subsequent design and development decisions.

## Super-Prompt
You are @elicitation-agent responsible for collaborating with the user to articulate the core project concept. Clearly define the main objectives, target audience, essential features, unique value proposition, and any competitive advantages. Document this in Project_Vision_Statement.md.

## MCP Tools Required
- edit_file: Create and update project vision documentation
- file_search: Locate existing vision statement or related documentation
- list_dir: Verify documentation structure

## Result We Want
- A clear project vision documented in Project_Vision_Statement.md, outlining objectives and target audience.

## Add to Brain
- Project Vision: Core concept, objectives, target audience, and success metrics

## Documentation & Templates
- [Project_Vision_Statement.md](mdc:01_Machine/04_Documentation/vision/Phase_1/01_User_Briefing/Project_Vision_Statement.md): Core project concept and objectives
- [Briefing_Summary.md](mdc:01_Machine/04_Documentation/vision/Phase_1/01_User_Briefing/Briefing_Summary.md): Briefing session summary

## Primary Responsible Agent
@elicitation-agent

## Supporting Agents
- @nlu-processor-agent
- @project-initiator-agent
- @market-research-agent
- @tech-spec-agent

## Agent Selection Criteria
The @elicitation-agent is chosen for its expertise in vision discovery, concept analysis, and document editing. Supporting agents provide NLU, project setup, industry context, and technical validation.

## Tasks (Summary)
- Elicit and document the core project vision, including objectives, target audience, and unique value proposition.

## Subtasks (Detailed)
### Subtask-00: Analyze Project.md (if present)
- **ID**: P01-T02-S00
- **Description**: Review and extract all relevant vision, goals, and context from `Project.md` to inform the vision elicitation process. Use this as a baseline and avoid redundant questions.
- **Agent Assignment**: @elicitation-agent
- **Documentation Links**: [Project.md](mdc:Project.md)
- **Steps**:
    1. Read `Project.md` and summarize key points (vision, goals, context).
    2. Use this information to pre-fill or guide the vision elicitation session.
    3. Only ask the user for missing or unclear information.

### Subtask-01: Core Concept Discovery
- **ID**: P01-T02-S01
- **Description**: Discover the main objectives, target audience, key features, and unique value proposition.
- **Agent Assignment**: @elicitation-agent
- **Documentation Links**:
  - [Project_Vision_Statement.md](mdc:01_Machine/04_Documentation/vision/Phase_1/01_User_Briefing/Project_Vision_Statement.md)
  - [Briefing_Summary.md](mdc:01_Machine/04_Documentation/vision/Phase_1/01_User_Briefing/Briefing_Summary.md)
  - [Elicitation Techniques Guide](mdc:01_Machine/04_Documentation/vision/Phase_1/01_User_Briefing/Elicitation_Techniques_Guide.md): General techniques for vision elicitation.
  - [Agent Collaboration Guide for P01-S01-T02](mdc:01_Machine/04_Documentation/vision/Phase_1/01_User_Briefing/Agent_Collaboration_Guide.md): Roles and workflow of agents for this task.
  - [MCP Tool Usage Guide for P01-S01-T02](mdc:01_Machine/04_Documentation/vision/Phase_1/01_User_Briefing/Tool_Usage_Guide.md): How to use MCP tools for this task.
- **Steps**:
    1. Collaborate with the user to articulate the core project concept, main objectives, target audience, essential features, unique value proposition, and competitive advantages.
    2. Document these in Project_Vision_Statement.md with distinct sections or bullet points for each required element.
- **Success Criteria**:
    - Project_Vision_Statement.md contains sections for objectives, target audience, key features (at least 3), unique value proposition, and competitive advantages.
    - Clarity Score: An automated NLP check (if available) indicates a minimum clarity score, or manual review fallback.
- **Max Retries**: 2
- **On Failure**: ESCALATE_TO_HUMAN (@project-lead) with logs and draft content.
- **Integration Points**: The project vision guides all subsequent design and development decisions.
- **Next Subtask**: P01-S01-T04-Success-Criteria-Definition.md

## Rollback Procedures
- If information is incomplete, schedule follow-up sessions.
- If requirements change, update documentation and re-validate with user.

## Integration Points
- Project vision guides all subsequent design and development decisions.

## Quality Gates
- Completeness Check: All required vision fields are filled.
- Clarity Validation: Vision is clearly articulated and unambiguous.
- Documentation Standards: Project_Vision_Statement.md is valid and structured.

## Success Criteria
- Clear project vision statement with objectives and target audience defined.

## Risk Mitigation
- Use structured questioning and follow-up sessions for incomplete information.
- Employ clarification techniques for unclear responses.

## Output Artifacts
- [Project_Vision_Statement.md](mdc:01_Machine/04_Documentation/vision/Phase_1/01_User_Briefing/Project_Vision_Statement.md): Core project concept and objectives

## Next Action
Initiate project vision elicitation and document findings.

## Post-Completion Action
Update Step.json and DNA.json to reflect task SUCCEEDED status and progress and referenced new output artifacts.
Upon completion, proceed to P01-S01-T04-Success-Criteria-Definition.md and update Step.json and DNA.json to reflect progress. 