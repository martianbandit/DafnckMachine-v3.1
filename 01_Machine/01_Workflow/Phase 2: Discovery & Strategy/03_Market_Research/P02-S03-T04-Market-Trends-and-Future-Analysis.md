---
phase: P02
step: S03
task: T04
task_id: P02-S03-T04
title: Market Trends and Future Analysis
previous_task: P02-S03-T03
next_task: P02-S03-T05
version: 3.1.0
source: Step.json
agent: "@market-research-agent"
orchestrator: "@uber-orchestrator-agent"
---

## Mission Statement
Analyze technology trends and forecast market evolution to inform long-term strategy and product roadmap development.

## Description
Analyze technology trends and forecast market evolution to inform long-term strategy and product roadmap development for DafnckMachine v3.1.

## Super-Prompt
"You are @market-research-agent responsible for conducting comprehensive market research for DafnckMachine v3.1. Your mission is to systematically analyze the industry landscape, competitive environment, customer segments, and market trends to provide strategic insights for product positioning and go-to-market strategy. Conduct thorough industry analysis, competitive intelligence gathering, customer segmentation, trend analysis, and strategic opportunity identification. Use both quantitative data and qualitative insights to create a comprehensive market understanding. Your research must be objective, data-driven, and provide actionable strategic recommendations for positioning, differentiation, and market entry. Document all findings in structured formats that support strategic decision-making and product development planning."

## MCP Tools Required
- web_search
- edit_file
- file_search
- list_dir

## Result We Want
- Technology trends analysis with innovation roadmap implications

## Add to Brain
- **Market Trends**: Growth patterns, emerging technologies, regulatory changes, and future projections

## Documentation & Templates
- [Technology_Trends_Report.md](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Technology_Trends_Report.md): Technology trends
- [Innovation_Timeline.json](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Innovation_Timeline.json): Innovation timeline
- [Market_Trends_Report.md](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Market_Trends_Report.md): Market trends and forecasting
- [Future_Scenarios_Analysis.json](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Future_Scenarios_Analysis.json): Scenario analysis

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
- Analyze technology trends and forecast market evolution to inform long-term strategy and product roadmap development.

## Subtasks (Detailed)
### Subtask-01: Technology Trends Assessment
- **ID**: P02-T04-S01
- **Description**: Analyze technology trends including emerging technologies, innovation patterns, adoption curves, disruption potential, and technology roadmaps.
- **Agent Assignment**: @technology-advisor-agent (tech-trends, innovation-analysis, technology-forecasting)
- **Documentation Links**: [Technology_Trends_Report.md](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Technology_Trends_Report.md), [Innovation_Timeline.json](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Innovation_Timeline.json)
- **Steps**:
    1. Research and analyze technology trends including emerging technologies, innovation patterns, and adoption curves (web_search)
    2. Document technology trends in Technology_Trends_Report.md and create Innovation_Timeline.json (edit_file)
- **Success Criteria**:
    - At least 5 major technology trends identified and analyzed
    - Innovation patterns and adoption curves documented
    - Technology_Trends_Report.md created with comprehensive trends analysis
    - Innovation_Timeline.json created with valid JSON structure
    - Report includes adoption projections and disruption potential assessment
- **On Failure**: ESCALATE_TO_HUMAN (@technology-strategist) with trend analysis data
- **Max Retries**: 2
- **Integration Points**: Technology trends inform product roadmap development and technical strategy formulation.
- **Next Subtask**: P02-T04-S02 (Market Evolution Forecasting)

### Subtask-02: Market Evolution Forecasting
- **ID**: P02-T04-S02
- **Description**: Forecast market evolution through scenario planning, trend extrapolation, disruption analysis, and future market structure predictions.
- **Agent Assignment**: @deep-research-agent (forecasting, scenario-analysis, trend-extrapolation)
- **Documentation Links**: [Market_Trends_Report.md](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Market_Trends_Report.md), [Future_Scenarios_Analysis.json](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Future_Scenarios_Analysis.json)
- **Steps**:
    1. Conduct market evolution forecasting using scenario planning and trend extrapolation methodologies (web_search)
    2. Document market forecasting in Market_Trends_Report.md and create Future_Scenarios_Analysis.json (edit_file)
- **Success Criteria**:
    - At least 3 future scenarios developed (optimistic, realistic, pessimistic)
    - Trend extrapolation and disruption analysis completed
    - Market_Trends_Report.md updated with evolution forecast
    - Future_Scenarios_Analysis.json created with valid JSON structure
    - Multiple scenarios documented with strategic implications
- **On Failure**: ESCALATE_TO_HUMAN (@strategic-forecaster) with forecasting analysis
- **Max Retries**: 2
- **Integration Points**: Market forecasting informs long-term strategy development and investment decision-making.
- **Next Subtask**: P02-T05-S01 (Regulatory Environment Assessment)

## Rollback Procedures
1. Insufficient Market Data: Expand research scope or use alternative data sources
2. Competitive Intelligence Gaps: Conduct additional competitor research or primary interviews
3. Customer Insight Limitations: Supplement with primary customer research or surveys
4. Strategic Misalignment: Revisit research objectives and refocus analysis
5. Recommendation Uncertainty: Conduct scenario analysis and sensitivity testing

## Integration Points
- Technology trends inform product roadmap development and technical strategy formulation.
- Market forecasting informs long-term strategy development and investment decision-making.

## Quality Gates
1. Research Depth: Comprehensive coverage of all market research dimensions
2. Data Reliability: High-quality sources and validated market data
3. Analysis Rigor: Systematic analysis with objective methodology
4. Strategic Relevance: Research findings directly support strategic decision-making
5. Actionability: Clear, implementable recommendations with specific next steps

## Success Criteria
- Technology trends analysis with innovation roadmap implications

## Risk Mitigation
- Data Quality Issues: Use multiple sources and cross-validate findings
- Market Volatility: Consider multiple scenarios and sensitivity analysis
- Competitive Intelligence: Use ethical research methods and public information
- Bias Prevention: Implement structured analysis frameworks and objective criteria
- Information Overload: Focus on strategic relevance and decision-critical insights

## Output Artifacts
- [Technology_Trends_Report.md](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Technology_Trends_Report.md)
- [Innovation_Timeline.json](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Innovation_Timeline.json)
- [Market_Trends_Report.md](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Market_Trends_Report.md)
- [Future_Scenarios_Analysis.json](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Future_Scenarios_Analysis.json)

## Next Action
Initiate technology trends and market evolution analysis with @market-research-agent

## Post-Completion Action
Upon successful completion of all subtasks, update @Step.json and @DNA.json to reflect the task's SUCCEEDED status and any associated progress or outcomes. 