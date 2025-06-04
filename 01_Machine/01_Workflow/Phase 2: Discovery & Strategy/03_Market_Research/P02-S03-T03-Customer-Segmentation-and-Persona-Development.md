---
phase: P02
step: S03
task: T03
task_id: P02-S03-T03
title: Customer Segmentation and Persona Development
previous_task: P02-S03-T02
next_task: P02-S03-T04
version: 3.1.0
source: Step.json
agent: "@market-research-agent"
orchestrator: "@uber-orchestrator-agent"
---

## Mission Statement
Conduct comprehensive customer segmentation analysis and develop detailed customer personas to guide product development and marketing strategy.

## Description
Conduct comprehensive customer segmentation analysis and develop detailed customer personas to guide product development and marketing strategy for DafnckMachine v3.1.

## Super-Prompt
"You are @market-research-agent responsible for conducting comprehensive market research for DafnckMachine v3.1. Your mission is to systematically analyze the industry landscape, competitive environment, customer segments, and market trends to provide strategic insights for product positioning and go-to-market strategy. Conduct thorough industry analysis, competitive intelligence gathering, customer segmentation, trend analysis, and strategic opportunity identification. Use both quantitative data and qualitative insights to create a comprehensive market understanding. Your research must be objective, data-driven, and provide actionable strategic recommendations for positioning, differentiation, and market entry. Document all findings in structured formats that support strategic decision-making and product development planning."

## MCP Tools Required
- web_search
- edit_file
- file_search
- list_dir

## Result We Want
- Customer segmentation with personas and journey mapping

## Add to Brain
- **Customer Insights**: Segmentation, personas, behaviors, needs, and decision-making processes

## Documentation & Templates
- [Customer_Segmentation_Analysis.md](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Customer_Segmentation_Analysis.md): Market segments, personas, and journey mapping
- [Segment_Attractiveness_Matrix.json](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Segment_Attractiveness_Matrix.json): Segment attractiveness
- [Customer_Personas.md](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Customer_Personas.md): Customer personas
- [Customer_Journey_Maps.json](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Customer_Journey_Maps.json): Journey maps

## Primary Responsible Agent
@market-research-agent

## Supporting Agents
- @deep-research-agent
- @idea-generation-agent
- @technology-advisor-agent
- @system-architect-agent

## Agent Selection Criteria
The Market Research Agent is chosen for its specialized capabilities in comprehensive market analysis, competitive intelligence, and strategic research. This agent excels at synthesizing complex market data, identifying strategic opportunities, and translating market insights into actionable business strategies.

## Tasks (Summary)
- Conduct comprehensive customer segmentation analysis and develop detailed customer personas to guide product development and marketing strategy.

## Subtasks (Detailed)
### Subtask-01: Market Segmentation Analysis
- **ID**: P02-T03-S01
- **Description**: Conduct market segmentation using demographic, psychographic, behavioral, and needs-based approaches with segment sizing and attractiveness assessment.
- **Agent Assignment**: @market-research-agent (segmentation-analysis, market-sizing, demographic-analysis)
- **Documentation Links**: [Customer_Segmentation_Analysis.md](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Customer_Segmentation_Analysis.md), [Segment_Attractiveness_Matrix.json](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Segment_Attractiveness_Matrix.json)
- **Steps**:
    1. Research and analyze market segmentation using demographic, psychographic, behavioral, and needs-based criteria (web_search)
    2. Document segmentation analysis in Customer_Segmentation_Analysis.md and create Segment_Attractiveness_Matrix.json (edit_file)
- **Success Criteria**:
    - At least 4 distinct market segments identified
    - Demographic, psychographic, and behavioral data collected for each segment
    - Customer_Segmentation_Analysis.md created with detailed segmentation analysis
    - Segment_Attractiveness_Matrix.json created with valid JSON structure
    - Analysis includes segment sizing and attractiveness assessment
- **On Failure**: ESCALATE_TO_HUMAN (@customer-insights-lead) with segmentation data
- **Max Retries**: 2
- **Integration Points**: Segmentation analysis informs target market selection and persona development strategy.
- **Next Subtask**: P02-T03-S02 (Customer Persona Development)

### Subtask-02: Customer Persona Development
- **ID**: P02-T03-S02
- **Description**: Develop detailed customer personas including demographics, behaviors, needs, pain points, decision-making processes, and journey mapping.
- **Agent Assignment**: @market-research-agent (persona-development, customer-insights, journey-mapping)
- **Documentation Links**: [Customer_Personas.md](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Customer_Personas.md), [Customer_Journey_Maps.json](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Customer_Journey_Maps.json)
- **Steps**:
    1. Develop detailed customer personas based on segmentation analysis including demographics, behaviors, needs, and pain points (edit_file)
    2. Document personas in Customer_Personas.md and create Customer_Journey_Maps.json with journey mapping (edit_file)
- **Success Criteria**:
    - At least 3 detailed customer personas created
    - Each persona includes demographics, behaviors, needs, pain points, and decision criteria
    - Customer_Personas.md created with detailed persona profiles
    - Customer_Journey_Maps.json created with valid JSON structure
    - Journey maps detail customer touchpoints and decision processes
- **On Failure**: ESCALATE_TO_HUMAN (@persona-specialist) with persona development data
- **Max Retries**: 2
- **Integration Points**: Personas guide product development priorities and marketing strategy formulation.
- **Next Subtask**: P02-T04-S01 (Technology Trends Assessment)

## Rollback Procedures
1. Insufficient Market Data: Expand research scope or use alternative data sources
2. Competitive Intelligence Gaps: Conduct additional competitor research or primary interviews
3. Customer Insight Limitations: Supplement with primary customer research or surveys
4. Strategic Misalignment: Revisit research objectives and refocus analysis
5. Recommendation Uncertainty: Conduct scenario analysis and sensitivity testing

## Integration Points
- Segmentation analysis informs target market selection and persona development strategy.
- Personas guide product development priorities and marketing strategy formulation.

## Quality Gates
1. Research Depth: Comprehensive coverage of all market research dimensions
2. Data Reliability: High-quality sources and validated market data
3. Analysis Rigor: Systematic analysis with objective methodology
4. Strategic Relevance: Research findings directly support strategic decision-making
5. Actionability: Clear, implementable recommendations with specific next steps

## Success Criteria
- Customer segmentation completed with detailed personas and journey maps

## Risk Mitigation
- Data Quality Issues: Use multiple sources and cross-validate findings
- Market Volatility: Consider multiple scenarios and sensitivity analysis
- Competitive Intelligence: Use ethical research methods and public information
- Bias Prevention: Implement structured analysis frameworks and objective criteria
- Information Overload: Focus on strategic relevance and decision-critical insights

## Output Artifacts
- [Customer_Segmentation_Analysis.md](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Customer_Segmentation_Analysis.md)
- [Segment_Attractiveness_Matrix.json](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Segment_Attractiveness_Matrix.json)
- [Customer_Personas.md](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Customer_Personas.md)
- [Customer_Journey_Maps.json](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Customer_Journey_Maps.json)

## Next Action
Initiate customer segmentation and persona development with @market-research-agent

## Post-Completion Action
Upon successful completion of all subtasks, update @Step.json and @DNA.json to reflect the task's SUCCEEDED status and any associated progress or outcomes. 