---
phase: P03
step: S06
task: T03
task_id: P03-S06-T03
title: Feature Scoring Ranking
previous_task: P03-S06-T02
next_task: P03-S06-T04
version: 3.1.0
source: Step.json
agent: "@prd-architect-agent"
orchestrator: "@uber-orchestrator-agent"
---

# Previous Task: P03-S06-T02-Feature-Analysis-Assessment.md
# Current Task: P03-S06-T03-Feature-Scoring-Ranking.md
# Next Task: P03-S06-T04-Value-vs-Effort-Analysis.md

## Workflow Metadata
- **Workflow-Step**: Feature Prioritization
- **TaskID**: P03-T03
- **Step ID**: S06
- **Version**: 3.1.0
- **LastUpdate**: 2025-01-27
- **Previous Task**: P03-S06-T02-Feature-Analysis-Assessment.md
- **Current Task**: P03-S06-T03-Feature-Scoring-Ranking.md
- **Next Task**: P03-S06-T04-Value-vs-Effort-Analysis.md

## Mission Statement
Apply scoring methodology to rank features using multi-criteria analysis and priority classification.

## Description
Score all features across evaluation criteria using established methodology, calculate weighted scores, normalize results, aggregate criteria scores, and generate comprehensive rankings. Classify feature priorities using MoSCoW categorization, priority tiers, feature grouping, theme organization, and release grouping.

## Super-Prompt
"You are @prd-architect-agent responsible for applying the scoring methodology to rank features for DafnckMachine v3.1. Your mission is to use multi-criteria analysis, calculate weighted and normalized scores, aggregate results, and generate comprehensive rankings. Classify priorities using MoSCoW and other frameworks, and document all results with clear rationale."

## MCP Tools Required
- edit_file

## Result We Want
- Complete feature scoring with transparent methodology and clear rankings for all features
- Clear priority classification with logical feature grouping and organization for development planning

## Add to Brain
- **Feature Scoring**: Weighted, normalized, and aggregated scores for all features
- **Priority Classification**: MoSCoW categories, priority tiers, feature grouping, release planning

## Documentation & Templates
- [Feature_Priority_Matrix.json](mdc:01_Machine/04_Documentation/Doc/Phase_3_Product_Definition_Design/06_Feature_Prioritization/Feature_Priority_Matrix.json)
- [Multi_Criteria_Scores.md](mdc:01_Machine/04_Documentation/Doc/Phase_3_Product_Definition_Design/06_Feature_Prioritization/Multi_Criteria_Scores.md)
- [MoSCoW_Analysis.json](mdc:01_Machine/04_Documentation/Doc/Phase_3_Product_Definition_Design/06_Feature_Prioritization/MoSCoW_Analysis.json)
- [Priority_Classification.md](mdc:01_Machine/04_Documentation/Doc/Phase_3_Product_Definition_Design/06_Feature_Prioritization/Priority_Classification.md)

## Primary Responsible Agent
@prd-architect-agent

## Supporting Agents
- @task-planning-agent

## Agent Selection Criteria
@prd-architect-agent leads feature scoring and ranking due to expertise in multi-criteria analysis and prioritization frameworks.

## Tasks (Summary)
- Score all features across evaluation criteria
- Classify feature priorities using MoSCoW and other frameworks

## Subtasks (Detailed)
### Subtask-01: Multi-Criteria Feature Scoring
- **ID**: P03-T03-S01
- **Description**: Score all features across evaluation criteria using established methodology, calculate weighted scores, normalize results, aggregate criteria scores, and generate comprehensive rankings.
- **Agent Assignment**: @prd-architect-agent
- **Documentation Links**:
  - [Feature_Priority_Matrix.json](mdc:01_Machine/04_Documentation/Doc/Phase_3_Product_Definition_Design/06_Feature_Prioritization/Feature_Priority_Matrix.json)
  - [Multi_Criteria_Scores.md](mdc:01_Machine/04_Documentation/Doc/Phase_3_Product_Definition_Design/06_Feature_Prioritization/Multi_Criteria_Scores.md)
- **Steps**:
  1. Apply scoring methodology to calculate weighted scores for all features across evaluation criteria (edit_file)
  2. Create feature priority matrix with detailed rankings and evaluation criteria scores (edit_file)
- **Success Criteria**:
  - Complete feature scoring completed with transparent methodology and clear rankings for all features.
  - Files exist and match content requirements.
- **Integration Points**: Feature scores provide objective foundation for all prioritization decisions and development planning activities.
- **Next Subtask**: P03-T03-S02 (Priority Classification & Grouping)

### Subtask-02: Priority Classification & Grouping
- **ID**: P03-T03-S02
- **Description**: Classify feature priorities using MoSCoW categorization, priority tiers, feature grouping, theme organization, and release grouping.
- **Agent Assignment**: @prd-architect-agent
- **Documentation Links**:
  - [MoSCoW_Analysis.json](mdc:01_Machine/04_Documentation/Doc/Phase_3_Product_Definition_Design/06_Feature_Prioritization/MoSCoW_Analysis.json)
  - [Priority_Classification.md](mdc:01_Machine/04_Documentation/Doc/Phase_3_Product_Definition_Design/06_Feature_Prioritization/Priority_Classification.md)
- **Steps**:
  1. Classify features using MoSCoW methodology and priority tiers based on scoring results (edit_file)
  2. Create MoSCoW analysis with detailed categorization and release grouping (edit_file)
- **Success Criteria**:
  - Clear priority classification completed with logical feature grouping and organization for development planning.
  - Files exist and match content requirements.
- **Integration Points**: Priority classification guides development planning, resource allocation, and release milestone definition.
- **Next Subtask**: P03-T04-S01 (Value-Effort Matrix Creation)

## Rollback Procedures
1. Priority Conflicts: Revisit evaluation criteria and stakeholder input for realignment
2. Resource Constraints: Adjust scope and timeline with stakeholder approval
3. Technical Blockers: Revise priorities based on updated technical feasibility assessment
4. Market Changes: Re-evaluate business value and adjust priorities accordingly
5. Stakeholder Disagreement: Facilitate additional alignment sessions and decision arbitration

## Integration Points
- Feature scores and priority classification guide all subsequent development planning and resource allocation.

## Quality Gates
1. Prioritization Objectivity: Scoring methodology is transparent, consistent, and bias-free
2. Strategic Alignment: Feature priorities align with business strategy and user needs
3. Technical Feasibility: Prioritization considers realistic technical constraints and capabilities
4. Resource Viability: Roadmap is achievable within available resources and timeline
5. Stakeholder Consensus: Strong agreement on priorities with clear commitment to execution

## Success Criteria
- Complete feature scoring with transparent methodology and clear rankings for all features
- Clear priority classification with logical feature grouping and organization for development planning

## Risk Mitigation
- Subjective Bias: Use objective scoring criteria and multiple evaluation perspectives
- Resource Overcommitment: Conservative capacity planning with realistic timeline buffers
- Changing Priorities: Flexible framework with change management and re-prioritization process
- Technical Blockers: Comprehensive dependency analysis and contingency planning
- Stakeholder Conflicts: Transparent decision process with clear rationale and conflict resolution

## Output Artifacts
- [Feature_Priority_Matrix.json](mdc:01_Machine/04_Documentation/vision/Phase_3/06_Feature_Prioritization/Feature_Priority_Matrix.json)
- [Multi_Criteria_Scores.md](mdc:01_Machine/04_Documentation/vision/Phase_3/06_Feature_Prioritization/Multi_Criteria_Scores.md)
- [MoSCoW_Analysis.json](mdc:01_Machine/04_Documentation/vision/Phase_3/06_Feature_Prioritization/MoSCoW_Analysis.json)
- [Priority_Classification.md](mdc:01_Machine/04_Documentation/vision/Phase_3/06_Feature_Prioritization/Priority_Classification.md)

## Next Action
Initiate feature scoring and ranking with @prd-architect-agent

## Post-Completion Action
Upon successful completion of all subtasks within this tactical task, ensure the @Step.json and @DNA.json files are updated to reflect its SUCCEEDED status and any associated progress or outcomes. 