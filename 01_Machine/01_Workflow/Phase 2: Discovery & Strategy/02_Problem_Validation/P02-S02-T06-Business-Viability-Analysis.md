---
phase: P02
step: S02
task: T06
task_id: P02-S02-T06
title: Business Viability Analysis
previous_task: P02-S02-T05
next_task: P02-S02-T07
version: 3.1.0
source: Step.json
agent: "@market-research-agent"
orchestrator: "@uber-orchestrator-agent"
---

## Mission Statement
Validate business model viability including revenue potential, cost structure, pricing strategies, and comprehensive risk assessment with mitigation strategies.

## Description
Analyze revenue potential, assess cost structure, evaluate pricing strategies, and identify monetization opportunities to validate business model viability. Conduct comprehensive risk assessment identifying market, technical, business, and competitive risks, and develop prioritized mitigation strategies.

## Super-Prompt
You are @market-research-agent responsible for business viability analysis. Your mission is to analyze revenue potential, cost structure, pricing strategies, and monetization opportunities, and to conduct comprehensive risk assessment and develop mitigation strategies. Document all findings and recommendations in structured formats that support strategic decision-making for subsequent workflow phases.

## MCP Tools Required
- edit_file: Create business viability, revenue model, and risk assessment documentation

## Result We Want
- Validated business model with revenue projections, cost analysis, and viability assessment completed.
- Complete risk assessment with prioritized mitigation strategies and contingency plans developed.

## Add to Brain
- Business Viability: Revenue model and cost structure analysis

## Documentation & Templates
- [Business_Viability_Analysis.json](mdc:01_Machine/04_Documentation/Doc/Phase_2/02_Problem_Validation/Business_Viability_Analysis.json): Business viability analysis
- [Revenue_Model.md](mdc:01_Machine/04_Documentation/Doc/Phase_2/02_Problem_Validation/Revenue_Model.md): Revenue model
- [Risk_Assessment_Matrix.json](mdc:01_Machine/04_Documentation/Doc/Phase_2/02_Problem_Validation/Risk_Assessment_Matrix.json): Risk assessment matrix
- [Mitigation_Strategies.md](mdc:01_Machine/04_Documentation/Doc/Phase_2/02_Problem_Validation/Mitigation_Strategies.md): Mitigation strategies

## Primary Responsible Agent
@market-research-agent

## Supporting Agents
- @technology-advisor-agent
- @system-architect-agent

## Agent Selection Criteria
The Market Research Agent is chosen for its specialized capabilities in business model, viability analysis, financial modeling, risk analysis, mitigation planning, and strategic planning.

## Tasks (Summary)
- Analyze revenue potential and monetization opportunities
- Assess cost structure and operational expenses
- Create business viability analysis with financial projections
- Identify and categorize all project risks
- Assess risk probability and impact levels
- Develop mitigation strategies and contingency plans

## Subtasks (Detailed)
### Subtask 1: Business Model Validation
- **ID**: P02-T06-S02
- **Description**: Analyze revenue potential, assess cost structure, evaluate pricing strategies, and identify monetization opportunities to validate business model viability.
- **Agent Assignment**: @market-research-agent
- **Documentation Links**: [Business_Viability_Analysis.json](mdc:01_Machine/04_Documentation/Doc/Phase_2/02_Problem_Validation/Business_Viability_Analysis.json), [Revenue_Model.md](mdc:01_Machine/04_Documentation/Doc/Phase_2/02_Problem_Validation/Revenue_Model.md)
- **Steps**:
    1. Analyze revenue potential and monetization opportunities (edit_file)
    2. Assess cost structure and operational expenses (edit_file)
    3. Create business viability analysis with financial projections (edit_file)
- **Success Criteria**:
    - File Created: 01_Machine/04_Documentation/vision/Phase_2/02_Problem_Validation/Revenue_Model.md
    - File Created: 01_Machine/04_Documentation/vision/Phase_2/02_Problem_Validation/Business_Viability_Analysis.json
    - File Content Contains: revenue_streams, pricing_models, monetization_strategies, cost_structure, operational_expenses, maintenance_costs, financial_projections, break_even_analysis, roi_calculations
- **Integration Points**: Business model validation supports investment decisions and resource allocation planning.
- **Next Subtask**: P02-S02-T07-Validation-Synthesis-Recommendation.md

### Subtask 2: Risk Assessment & Mitigation
- **ID**: P02-T06-S03
- **Description**: Conduct comprehensive risk assessment identifying market, technical, business, and competitive risks, and develop prioritized mitigation strategies.
- **Agent Assignment**: @market-research-agent
- **Documentation Links**: [Risk_Assessment_Matrix.json](mdc:01_Machine/04_Documentation/Doc/Phase_2/02_Problem_Validation/Risk_Assessment_Matrix.json), [Mitigation_Strategies.md](mdc:01_Machine/04_Documentation/Doc/Phase_2/02_Problem_Validation/Mitigation_Strategies.md)
- **Steps**:
    1. Identify and categorize all project risks (edit_file)
    2. Assess risk probability and impact levels (edit_file)
    3. Develop mitigation strategies and contingency plans (edit_file)
- **Success Criteria**:
    - File Created: 01_Machine/04_Documentation/Doc/Phase_2/02_Problem_Validation/Risk_Assessment_Matrix.json
    - File Created: 01_Machine/04_Documentation/Doc/Phase_2/02_Problem_Validation/Mitigation_Strategies.md
    - File Content Contains: market_risks, technical_risks, business_risks, competitive_risks, risk_probability, impact_assessment, risk_priority, mitigation_strategies, contingency_plans, risk_monitoring
- **Integration Points**: Risk assessment informs project planning, resource allocation, and contingency strategies for implementation.
- **Next Subtask**: P02-S02-T07-Validation-Synthesis-Recommendation.md

## Rollback Procedures
- Explore alternative business models or market segments if business unviability is found

## Integration Points
- Business model validation supports investment decisions and resource allocation planning
- Risk assessment informs project planning and contingency strategies

## Quality Gates
- Research rigor: Use of multiple sources and methodologies
- Data quality: Reliable, up-to-date financial and risk data
- Analysis depth: Comprehensive business and risk assessment
- Objectivity: Unbiased, evidence-based conclusions
- Actionability: Clear, actionable outputs

## Success Criteria
- Business viability assessed with revenue model and cost structure
- Risk assessment completed with prioritized mitigation strategies

## Risk Mitigation
- Use multiple research sources and validation methods
- Escalate to human if analysis fails after 3 retries

## Output Artifacts
- [Business_Viability_Analysis.json](mdc:01_Machine/04_Documentation/Doc/Phase_2/02_Problem_Validation/Business_Viability_Analysis.json)
- [Revenue_Model.md](mdc:01_Machine/04_Documentation/Doc/Phase_2/02_Problem_Validation/Revenue_Model.md)
- [Risk_Assessment_Matrix.json](mdc:01_Machine/04_Documentation/Doc/Phase_2/02_Problem_Validation/Risk_Assessment_Matrix.json)
- [Mitigation_Strategies.md](mdc:01_Machine/04_Documentation/Doc/Phase_2/02_Problem_Validation/Mitigation_Strategies.md)

## Next Action
Initiate business model validation and risk assessment with @market-research-agent

## Post-Completion Action
Update Step.json and DNA.json to reflect task SUCCEEDED status and progress and referenced new output artifacts.
Upon completion, proceed to P02-S02-T07-Context-and-Market-Analysis.md and update Step.json and DNA.json to reflect progress. 