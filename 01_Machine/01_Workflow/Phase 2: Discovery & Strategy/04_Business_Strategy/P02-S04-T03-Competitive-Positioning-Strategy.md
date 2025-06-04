---
phase: P02
step: S04
task: T03
task_id: P02-S04-T03
title: Competitive Positioning Strategy
previous_task: P02-S04-T02
next_task: P02-S04-T04
version: 3.1.0
source: Step.json
agent: "@market-research-agent"
orchestrator: "@uber-orchestrator-agent"
---

## Mission Statement
Develop competitive positioning strategy with differentiation framework and competitive response planning for DafnckMachine v3.1.

## Description
This task develops a competitive positioning strategy, including differentiation framework, competitive advantages, and response planning. The strategy will be validated against competitive intelligence and market analysis.

## Super-Prompt
"You are @market-research-agent. Your mission is to develop a competitive positioning strategy for DafnckMachine v3.1, including differentiation framework, competitive advantages, and response planning. Validate all strategies against competitive intelligence and document in structured formats."

## MCP Tools Required
- edit_file: Create positioning framework documentation
- file_search: Access competitive analysis data
- web_search: Research competitor strategies
- list_dir: Organize positioning strategy documents

## Result We Want
- Differentiation strategy with sustainable competitive advantages
- Competitive response plan with threat assessment

## Add to Brain
- Differentiation Framework
- Competitive Advantages
- Competitive Response Plan

## Documentation & Templates
- [Strategic_Positioning_Framework.md](mdc:01_Machine/04_Documentation/Doc/Phase_2/04_Business_Strategy/Strategic_Positioning_Framework.md)
- [Differentiation_Framework.json](mdc:01_Machine/04_Documentation/Doc/Phase_2/04_Business_Strategy/Differentiation_Framework.json)
- [Competitive_Response_Plan.md](mdc:01_Machine/04_Documentation/Doc/Phase_2/04_Business_Strategy/Competitive_Response_Plan.md)
- [Risk_Assessment_Matrix.json](mdc:01_Machine/04_Documentation/Doc/Phase_2/04_Business_Strategy/Risk_Assessment_Matrix.json)

## Supporting Agents
- @technology-advisor-agent
- @system-architect-agent
- @marketing-strategy-orchestrator

## Agent Selection Criteria
The @market-research-agent is chosen for its expertise in differentiation, competitive analysis, and strategic planning.

## Tasks (Summary)
- Develop differentiation strategy
- Plan competitive responses

## Subtasks (Detailed)
### Subtask 1: Differentiation Strategy Development
- **ID**: P02-T03-S01
- **Description**: Develop differentiation strategy with unique value propositions and competitive advantages.
- **Agent**: @market-research-agent
- **Documentation Links**:
  - [Strategic_Positioning_Framework.md](mdc:01_Machine/04_Documentation/Doc/Phase_2/04_Business_Strategy/Strategic_Positioning_Framework.md)
  - [Differentiation_Framework.json](mdc:01_Machine/04_Documentation/Doc/Phase_2/04_Business_Strategy/Differentiation_Framework.json)
- **Steps**:
  1. Develop differentiation strategy (edit_file)
  2. Validate against competitive analysis data (file_search)
- **Success Criteria**:
  - File exists: Strategic_Positioning_Framework.md
  - Minimum 3 differentiation factors identified
  - Competitive advantage sustainability analysis
  - Differentiation validated against competitor analysis

### Subtask 2: Competitive Response Planning
- **ID**: P02-T03-S02
- **Description**: Plan competitive responses including threat assessment and strategic options.
- **Agent**: @market-research-agent
- **Documentation Links**:
  - [Competitive_Response_Plan.md](mdc:01_Machine/04_Documentation/Doc/Phase_2/04_Business_Strategy/Competitive_Response_Plan.md)
  - [Risk_Assessment_Matrix.json](mdc:01_Machine/04_Documentation/Doc/Phase_2/04_Business_Strategy/Risk_Assessment_Matrix.json)
- **Steps**:
  1. Develop competitive response framework (edit_file)
  2. Design competitive intelligence monitoring framework (edit_file)
- **Success Criteria**:
  - File exists: Competitive_Response_Plan.md
  - Threat assessment matrix with risk levels
  - Both defensive and offensive strategies documented
  - Competitive monitoring framework included
  - Response triggers and escalation procedures defined

## Rollback Procedures
- Strengthen differentiation strategy and competitive advantages
- Update response plan based on new threats

## Integration Points
- Differentiation strategy guides product and marketing
- Competitive planning informs product roadmap

## Quality Gates
- Competitive advantage demonstrated
- Strategic coherence with market analysis

## Success Criteria
- [ ] Differentiation strategy documented
- [ ] Competitive response plan completed

## Risk Mitigation
- Multiple differentiation strategies
- Ongoing competitive intelligence monitoring

## Output Artifacts
- [Strategic_Positioning_Framework.md](mdc:01_Machine/04_Documentation/vision/Phase_2/04_Business_Strategy/Strategic_Positioning_Framework.md)
- [Competitive_Response_Plan.md](mdc:01_Machine/04_Documentation/vision/Phase_2/04_Business_Strategy/Competitive_Response_Plan.md)

## Next Action
Initiate differentiation strategy development with @market-research-agent

## Post-Completion Action
Update Step.json and DNA.json to reflect SUCCEEDED status for this task and its outcomes. 