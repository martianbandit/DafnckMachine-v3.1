---
phase: P03
step: S06
task: T04
task_id: P03-S06-T04
title: Value vs Effort Analysis
previous_task: P03-S06-T03
next_task: P03-S07-T01
version: 3.1.0
source: Step.json
agent: "@prd-architect-agent"
orchestrator: "@uber-orchestrator-agent"
---

## Mission Statement
Create value-effort analysis to identify quick wins, strategic investments, and optimization opportunities.

## Description
Create a comprehensive value-effort matrix plotting features by business value and development effort to identify quick wins, major investments, and low-value features. Identify and analyze strategic opportunities including quick wins for immediate value delivery, strategic investments for long-term advantage, and optimization opportunities.

## Super-Prompt
"You are @prd-architect-agent responsible for value-effort analysis for DafnckMachine v3.1. Your mission is to create a value-effort matrix, identify quick wins and strategic investments, and document all findings with clear rationale and supporting data."

## MCP Tools Required
- edit_file

## Result We Want
- Comprehensive value-effort analysis completed with strategic feature positioning and optimization opportunities identified
- Clear identification of quick wins and strategic investment opportunities with actionable implementation guidance

## Add to Brain
- **Value-Effort Matrix**: Feature positioning based on business value and development effort
- **Quick Wins & Strategic Investments**: Immediate value delivery and long-term advantage opportunities

## Documentation & Templates
- [Value_vs_Effort_Matrix.json](mdc:01_Machine/04_Documentation/Doc/Phase_3_Product_Definition_Design/06_Feature_Prioritization/Value_vs_Effort_Matrix.json)
- [Quick_Wins_Analysis.md](mdc:01_Machine/04_Documentation/Doc/Phase_3_Product_Definition_Design/06_Feature_Prioritization/Quick_Wins_Analysis.md)
- [Strategic_Investment_Plan.md](mdc:01_Machine/04_Documentation/Doc/Phase_3_Product_Definition_Design/06_Feature_Prioritization/Strategic_Investment_Plan.md)
- [Quick_Wins_Roadmap.json](mdc:01_Machine/04_Documentation/Doc/Phase_3_Product_Definition_Design/06_Feature_Prioritization/Quick_Wins_Roadmap.json)

## Primary Responsible Agent
@prd-architect-agent

## Supporting Agents
- @task-planning-agent

## Agent Selection Criteria
@prd-architect-agent leads value-effort analysis and strategic opportunity identification due to expertise in prioritization and planning.

## Tasks (Summary)
- Create value-effort matrix plotting features by business value and development effort
- Identify quick wins and strategic investment opportunities

## Subtasks (Detailed)
### Subtask-01: Value-Effort Matrix Creation
- **ID**: P03-T04-S01
- **Description**: Create comprehensive value-effort matrix plotting features by business value and development effort to identify quick wins, major investments, and low-value features.
- **Agent Assignment**: @prd-architect-agent
- **Documentation Links**:
  - [Value_vs_Effort_Matrix.json](mdc:01_Machine/04_Documentation/Doc/Phase_3_Product_Definition_Design/06_Feature_Prioritization/Value_vs_Effort_Matrix.json)
  - [Quick_Wins_Analysis.md](mdc:01_Machine/04_Documentation/Doc/Phase_3_Product_Definition_Design/06_Feature_Prioritization/Quick_Wins_Analysis.md)
- **Steps**:
  1. Create value-effort matrix plotting features by business value and development effort (edit_file)
  2. Generate value-effort matrix JSON with feature positioning and strategic categorization (edit_file)
- **Success Criteria**:
  - Comprehensive value-effort analysis completed with strategic feature positioning and optimization opportunities identified.
  - Files exist and match content requirements.
- **Integration Points**: Value-effort matrix optimizes development ROI and guides resource utilization decisions throughout the project.
- **Next Subtask**: P03-T04-S02 (Quick Wins & Strategic Investments)

### Subtask-02: Quick Wins & Strategic Investments
- **ID**: P03-T04-S02
- **Description**: Identify and analyze strategic opportunities including quick wins for immediate value delivery, strategic investments for long-term advantage, and optimization opportunities.
- **Agent Assignment**: @prd-architect-agent
- **Documentation Links**:
  - [Strategic_Investment_Plan.md](mdc:01_Machine/04_Documentation/Doc/Phase_3_Product_Definition_Design/06_Feature_Prioritization/Strategic_Investment_Plan.md)
  - [Quick_Wins_Roadmap.json](mdc:01_Machine/04_Documentation/Doc/Phase_3_Product_Definition_Design/06_Feature_Prioritization/Quick_Wins_Roadmap.json)
- **Steps**:
  1. Identify quick wins and strategic investment opportunities based on value-effort analysis (edit_file)
  2. Create quick wins roadmap with immediate value delivery opportunities (edit_file)
- **Success Criteria**:
  - Clear identification of quick wins and strategic investment opportunities with actionable implementation guidance.
  - Files exist and match content requirements.
- **Integration Points**: Strategic analysis guides development sequencing, resource allocation, and value delivery optimization throughout the project lifecycle.
- **Next Subtask**: P03-T05-S01 (Feature Dependency Mapping)

## Rollback Procedures
1. Priority Conflicts: Revisit evaluation criteria and stakeholder input for realignment
2. Resource Constraints: Adjust scope and timeline with stakeholder approval
3. Technical Blockers: Revise priorities based on updated technical feasibility assessment
4. Market Changes: Re-evaluate business value and adjust priorities accordingly
5. Stakeholder Disagreement: Facilitate additional alignment sessions and decision arbitration

## Integration Points
- Value-effort matrix and quick wins analysis guide development sequencing and resource allocation.

## Quality Gates
1. Prioritization Objectivity: Scoring methodology is transparent, consistent, and bias-free
2. Strategic Alignment: Feature priorities align with business strategy and user needs
3. Technical Feasibility: Prioritization considers realistic technical constraints and capabilities
4. Resource Viability: Roadmap is achievable within available resources and timeline
5. Stakeholder Consensus: Strong agreement on priorities with clear commitment to execution

## Success Criteria
- Comprehensive value-effort analysis completed with strategic feature positioning and optimization opportunities identified
- Clear identification of quick wins and strategic investment opportunities with actionable implementation guidance

## Risk Mitigation
- Subjective Bias: Use objective scoring criteria and multiple evaluation perspectives
- Resource Overcommitment: Conservative capacity planning with realistic timeline buffers
- Changing Priorities: Flexible framework with change management and re-prioritization process
- Technical Blockers: Comprehensive dependency analysis and contingency planning
- Stakeholder Conflicts: Transparent decision process with clear rationale and conflict resolution

## Output Artifacts
- [Value_vs_Effort_Matrix.json](mdc:01_Machine/04_Documentation/vision/Phase_3/06_Feature_Prioritization/Value_vs_Effort_Matrix.json)
- [Quick_Wins_Analysis.md](mdc:01_Machine/04_Documentation/vision/Phase_3/06_Feature_Prioritization/Quick_Wins_Analysis.md)
- [Strategic_Investment_Plan.md](mdc:01_Machine/04_Documentation/vision/Phase_3/06_Feature_Prioritization/Strategic_Investment_Plan.md)
- [Quick_Wins_Roadmap.json](mdc:01_Machine/04_Documentation/vision/Phase_3/06_Feature_Prioritization/Quick_Wins_Roadmap.json)

## Next Action
Initiate value-effort analysis and quick wins identification with @prd-architect-agent

## Post-Completion Action
Upon successful completion of all subtasks within this tactical task, ensure the @Step.json and @DNA.json files are updated to reflect its SUCCEEDED status and any associated progress or outcomes. 