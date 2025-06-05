---
phase: P01
step: S01
task: T01
task_id: P01-S01-T01
title: User Profile Development
previous_task: PHASE0-INIT-001
next_task: P01-S01-T02
version: 3.1.0
source: Step.json
agent: "@nlu-processor-agent"
orchestrator: "@uber-orchestrator-agent"
---
## Output Artifacts Checklist
- [ ] 01_Machine/04_Documentation/vision/Phase_1/01_User_Briefing/User_Profile.json — User_Profile.json: Complete user background and preferences (missing)

## Mission Statement
Develop a comprehensive profile of the user to tailor agent interaction and project approach for DafnckMachine v3.1.

## Description
This task focuses on assessing the user's technical expertise, industry experience, and preferred working styles. The resulting user profile will inform agent selection throughout the project.

## Super-Prompt
You are @nlu-processor-agent.json responsible for conducting a structured elicitation session to gather information on the user's technical expertise, industry experience, previous project involvement, and preferred working styles. Document all findings in structured formats and validate completeness.

## MCP Tools Required
- edit_file: Create and update user profile documentation
- file_search: Locate existing user profile or related documentation
- list_dir: Verify documentation structure

## Result We Want
- Complete user profile with background, expertise, and preferences documented in User_Profile.json

## Add to Brain
- User Profile: Background, experience level, technical expertise, and preferences

## Documentation & Templates
- [User_Profile.json](mdc:01_Machine/04_Documentation/vision/Phase_1/01_User_Briefing/User_Profile.json): User background, expertise, and preferences
- [Briefing_Summary.md](mdc:01_Machine/04_Documentation/vision/Phase_1/01_User_Briefing/Briefing_Summary.md): Briefing session summary

## Primary Responsible Agent
@"nlu-processor-agent.json"

## Supporting Agents
- @elicitation-agent
- @project-initiator-agent
- @market-research-agent
- @tech-spec-agent

## Agent Selection Criteria
The @nlu-processor-agent is chosen for its expertise in user profiling, background analysis, and document editing. Supporting agents provide advanced requirement gathering, project setup, industry context, and technical validation.

## Tasks (Summary)
- Develop a comprehensive user profile to inform agent selection strategies.

## Subtasks (Detailed)
### Subtask-01: Background Assessment
- **ID**: P01-T01-S01
- **Description**: Assess user's technical expertise, industry experience, and preferred working styles.
- **Agent Assignment**: @nlu-processor-agent
- **Documentation Links**:
  - [User_Profile.json](mdc:01_Machine/04_Documentation/vision/Phase_1/01_User_Briefing/User_Profile.json)
  - [Briefing_Summary.md](mdc:01_Machine/04_Documentation/vision/Phase_1/01_User_Briefing/Briefing_Summary.md)
- **Steps**:
    1. Engage the user to gather detailed information on their technical background, industry experience, and preferred project methodologies. Document responses thoroughly.
    2. Conduct a structured elicitation session (e.g., via interactive prompt or pre-defined questionnaire) to gather information on technical expertise, industry experience, previous projects, and working styles.
- **Success Criteria**:
    - User_Profile.json contains entries for technical_expertise, industry_experience, previous_projects, and working_styles, each with at least one valid value.
    - User_Profile.json is a valid JSON document and adheres to the expected schema for user profiles.
- **Max Retries**: 2
- **On Failure**: ESCALATE_TO_HUMAN (@nlu-lead) with full logs and status.
- **Integration Points**: The completed user profile will be used to inform subsequent agent selection throughout the project.
- **Next Subtask**: P01-S01-T02-Communication-Preferences.md

## Rollback Procedures
- If information is incomplete, schedule follow-up briefing sessions.
- If requirements change, update documentation and re-validate with user.

## Integration Points
- User profile feeds into agent selection strategies for all subsequent phases.

## Quality Gates
- Completeness Check: All required user profile fields are filled.
- Clarity Validation: Information is clearly articulated and unambiguous.
- Documentation Standards: User_Profile.json is valid and structured.

## Success Criteria
- Complete user profile with background, expertise, and preferences documented.

## Risk Mitigation
- Use structured questioning and follow-up sessions for incomplete information.
- Employ clarification techniques for unclear responses.

## Next Action
Initiate user background assessment and begin structured elicitation session.

## Post-Completion Action
Update Step.json and DNA.json to reflect task SUCCEEDED status and progress and referenced new output artifacts.
Upon completion, proceed to P01-S01-T02-Communication-Preferences.md and update Step.json and DNA.json to reflect progress. 