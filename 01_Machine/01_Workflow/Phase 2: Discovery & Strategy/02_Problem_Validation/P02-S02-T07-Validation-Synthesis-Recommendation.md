---
phase: P02
step: S02
task: T07
task_id: P02-S02-T07
title: Validation Synthesis Recommendation
previous_task: P02-S02-T06
next_task: P02-S03-T01
version: 3.1.0
source: Step.json
agent: "@market-research-agent"
orchestrator: "@uber-orchestrator-agent"
---

## Mission Statement
Synthesize all validation data, analyze findings against success criteria, and develop strategic recommendations with clear go/no-go decision and next steps.

## Description
Consolidate all research findings, analyze validation metrics, identify key insights, and assess overall problem validation success against defined criteria. Develop strategic recommendations based on validation synthesis, provide clear go/no-go decision with supporting rationale, and define actionable next steps.

## Super-Prompt
You are @market-research-agent responsible for validation synthesis and strategic recommendation. Your mission is to consolidate all research findings, analyze validation metrics, identify key insights, and develop evidence-based recommendations with clear go/no-go decision and next steps. Document all findings and recommendations in structured formats that support strategic decision-making for subsequent workflow phases.

## MCP Tools Required
- file_search: Gather and review all validation documents and research outputs
- edit_file: Create data analysis, validation summary, and strategic recommendation documentation

## Result We Want
- Comprehensive validation synthesis completed with key insights and metrics analysis documented.
- Clear strategic recommendation with actionable next steps and success criteria defined for Phase 3 execution.

## Add to Brain
- Validation Metrics: Key indicators and success criteria for problem validation

## Documentation & Templates
- [Validation_Summary.md](mdc:01_Machine/04_Documentation/vision/Phase_2/02_Problem_Validation/Validation_Summary.md): Validation summary
- [Data_Analysis_Report.json](mdc:01_Machine/04_Documentation/Doc/Phase_2/02_Problem_Validation/Data_Analysis_Report.json): Data analysis report
- [Strategic_Recommendation.md](mdc:01_Machine/04_Documentation/vision/Phase_2/02_Problem_Validation/Strategic_Recommendation.md): Strategic recommendation
- [Next_Steps_Plan.json](mdc:01_Machine/04_Documentation/vision/Phase_2/02_Problem_Validation/Next_Steps_Plan.json): Next steps plan

## Primary Responsible Agent
@market-research-agent

## Supporting Agents
- @technology-advisor-agent
- @system-architect-agent

## Agent Selection Criteria
The Market Research Agent is chosen for its specialized capabilities in data synthesis, analysis consolidation, insight generation, recommendation development, and strategic planning.

## Tasks (Summary)
- Consolidate all validation research findings and data
- Analyze validation metrics and assess success criteria
- Create comprehensive validation synthesis document
- Develop go/no-go recommendation with supporting evidence
- Define strategic next steps and optimization opportunities
- Create actionable next steps plan with success criteria

## Subtasks (Detailed)
### Subtask 1: Data Analysis & Synthesis
- **ID**: P02-T07-S02
- **Description**: Consolidate all research findings, analyze validation metrics, identify key insights, and assess overall problem validation success against defined criteria.
- **Agent Assignment**: @market-research-agent
- **Documentation Links**: [Validation_Summary.md](mdc:01_Machine/04_Documentation/vision/Phase_2/02_Problem_Validation/Validation_Summary.md), [Data_Analysis_Report.json](mdc:01_Machine/04_Documentation/Doc/Phase_2/02_Problem_Validation/Data_Analysis_Report.json)
- **Steps**:
    1. Consolidate all validation research findings and data (file_search)
    2. Analyze validation metrics and assess success criteria (edit_file)
    3. Create comprehensive validation synthesis document (edit_file)
- **Success Criteria**:
    - Output Contains: validation data, research findings
    - File Created: 01_Machine/04_Documentation/vision/Phase_2/02_Problem_Validation/Data_Analysis_Report.json
    - File Created: 01_Machine/04_Documentation/vision/Phase_2/02_Problem_Validation/Validation_Summary.md
    - File Content Contains: metrics_analysis, success_assessment, key_insights, validation_synthesis, evidence_summary, conclusions
- **Integration Points**: Synthesis provides evidence foundation for strategic recommendations and decision-making.
- **Next Subtask**: P03-S01-T01

### Subtask 2: Strategic Recommendation
- **ID**: P02-T07-S02
- **Description**: Develop strategic recommendations based on validation synthesis, provide clear go/no-go decision with supporting rationale, and define actionable next steps.
- **Agent Assignment**: @market-research-agent
- **Documentation Links**: [Strategic_Recommendation.md](mdc:01_Machine/04_Documentation/vision/Phase_2/02_Problem_Validation/Strategic_Recommendation.md), [Next_Steps_Plan.json](mdc:01_Machine/04_Documentation/vision/Phase_2/02_Problem_Validation/Next_Steps_Plan.json)
- **Steps**:
    1. Develop go/no-go recommendation with supporting evidence (edit_file)
    2. Define strategic next steps and optimization opportunities (edit_file)
    3. Create actionable next steps plan with success criteria (edit_file)
- **Success Criteria**:
    - File Created: 01_Machine/04_Documentation/vision/Phase_2/02_Problem_Validation/Strategic_Recommendation.md
    - File Created: 01_Machine/04_Documentation/vision/Phase_2/02_Problem_Validation/Next_Steps_Plan.json
    - File Content Contains: go_no_go_decision, supporting_rationale, evidence_base, next_steps, optimization_opportunities, strategic_priorities, action_items, timelines, success_criteria, responsibilities
- **Integration Points**: Recommendations guide Phase 3 planning, resource allocation, and strategic direction for product development.
- **Next Subtask**: P03-S01-T01

## Rollback Procedures
- Revisit validation synthesis and recommendations if evidence is insufficient

## Integration Points
- Synthesis provides evidence foundation for strategic recommendations and decision-making
- Recommendations guide Phase 3 planning and resource allocation

## Quality Gates
- Research rigor: Use of multiple sources and methodologies
- Data quality: Reliable, up-to-date validation and recommendation data
- Analysis depth: Comprehensive synthesis and recommendation
- Objectivity: Unbiased, evidence-based conclusions
- Actionability: Clear, actionable outputs

## Success Criteria
- Complete validation synthesis with data-driven recommendations
- Clear go/no-go recommendation with supporting evidence
- Strategic next steps defined for Phase 3 execution

## Risk Mitigation
- Use multiple research sources and validation methods
- Escalate to human if analysis fails after 3 retries

## Output Artifacts
- [Validation_Summary.md](mdc:01_Machine/04_Documentation/vision/Phase_2/02_Problem_Validation/Validation_Summary.md)
- [Data_Analysis_Report.json](mdc:01_Machine/04_Documentation/Doc/Phase_2/02_Problem_Validation/Data_Analysis_Report.json)
- [Strategic_Recommendation.md](mdc:01_Machine/04_Documentation/vision/Phase_2/02_Problem_Validation/Strategic_Recommendation.md)
- [Next_Steps_Plan.json](mdc:01_Machine/04_Documentation/vision/Phase_2/02_Problem_Validation/Next_Steps_Plan.json)

## Next Action
Initiate validation synthesis and strategic recommendation with @market-research-agent

## Post-Completion Action
Update Step.json and DNA.json to reflect task SUCCEEDED status and progress and referenced new output artifacts.
Upon completion, proceed to P02-S01-T01-Context-and-Market-Analysis.md and update Step.json and DNA.json to reflect progress. 