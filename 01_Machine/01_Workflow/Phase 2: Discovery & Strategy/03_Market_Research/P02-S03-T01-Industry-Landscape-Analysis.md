---
phase: P02
step: S03
task: T01
task_id: P02-S03-T01
title: Industry Landscape Analysis
previous_task: P02-S01-T07
next_task: P02-S03-T02
version: 3.1.0
source: Step.json
agent: "@market-research-agent"
orchestrator: "@uber-orchestrator-agent"
---
## Output Artifacts Checklist
- [ ] 01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Industry_Analysis_Report.md — Industry_Analysis_Report.md: Industry_Analysis_Report.md (missing)
- [ ] 01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Market_Structure_Map.json — Market_Structure_Map.json: Market_Structure_Map.json (missing)

## Mission Statement
Conduct comprehensive analysis of industry structure, market dynamics, and competitive landscape to establish foundational market understanding.

## Description
Conduct comprehensive market research to understand industry dynamics, competitive landscape, customer segments, market trends, and strategic opportunities that will inform product positioning, go-to-market strategy, and competitive differentiation for DafnckMachine v3.1.

## Super-Prompt
"You are @market-research-agent responsible for conducting comprehensive market research for DafnckMachine v3.1. Your mission is to systematically analyze the industry landscape, competitive environment, customer segments, and market trends to provide strategic insights for product positioning and go-to-market strategy. Conduct thorough industry analysis, competitive intelligence gathering, customer segmentation, trend analysis, and strategic opportunity identification. Use both quantitative data and qualitative insights to create a comprehensive market understanding. Your research must be objective, data-driven, and provide actionable strategic recommendations for positioning, differentiation, and market entry. Document all findings in structured formats that support strategic decision-making and product development planning."

## MCP Tools Required
- web_search
- edit_file
- file_search
- list_dir

## Result We Want
- Comprehensive industry analysis with market dynamics and growth drivers
- Detailed competitive intelligence with positioning opportunities
- Customer segmentation with personas and journey mapping
- Market trends analysis with future projections and implications
- Strategic positioning framework with differentiation strategies
- Go-to-market insights with channel and pricing recommendations

## Add to Brain
- **Industry Landscape**: Market structure, key players, value chains, and ecosystem dynamics
- **Competitive Intelligence**: Competitor profiles, strategies, strengths, weaknesses, and market positioning
- **Customer Insights**: Segmentation, personas, behaviors, needs, and decision-making processes
- **Market Trends**: Growth patterns, emerging technologies, regulatory changes, and future projections
- **Strategic Opportunities**: Market gaps, positioning opportunities, and competitive advantages
- **Market Entry Strategy**: Channel analysis, pricing models, and go-to-market recommendations

## Documentation & Templates
- [Industry_Analysis_Report.md](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Industry_Analysis_Report.md): Comprehensive industry landscape and dynamics
- [Market_Structure_Map.json](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Market_Structure_Map.json): Market structure mapping

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
- Conduct comprehensive analysis of industry structure, market dynamics, and competitive landscape to establish foundational market understanding.

## Subtasks (Detailed)
### Subtask-01: Market Structure Analysis
- **ID**: P02-T01-S01
- **Description**: Analyze industry structure including key market segments, value chain analysis, industry consolidation trends, and market maturity assessment.
- **Agent Assignment**: @market-research-agent (industry-analysis, market-structure, data-synthesis)
- **Documentation Links**: [Industry_Analysis_Report.md](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Industry_Analysis_Report.md), [Market_Structure_Map.json](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Market_Structure_Map.json)
- **Steps**:
    1. Research and analyze industry structure including market segments, value chain components, and consolidation trends (web_search)
    2. Document comprehensive industry structure analysis in Industry_Analysis_Report.md and create structured Market_Structure_Map.json (edit_file)
- **Success Criteria**:
    - Industry data collected from at least 5 authoritative sources
    - Analysis covers market segments, value chain, and consolidation trends
    - Industry_Analysis_Report.md created and contains market structure analysis
    - Market_Structure_Map.json created with valid JSON structure
    - Report contains sections on market segments, value chain, consolidation trends, and maturity assessment
- **On Failure**: ESCALATE_TO_HUMAN (@market-research-lead) with detailed analysis logs
- **Max Retries**: 2
- **Integration Points**: Industry structure analysis provides foundation for competitive positioning and market entry strategy development.
- **Next Subtask**: P02-T01-S02 (Market Dynamics Assessment)

### Subtask-02: Market Dynamics Assessment
- **ID**: P02-T01-S02
- **Description**: Assess market dynamics including growth drivers, market forces, cyclical patterns, disruption factors, and regulatory influences.
- **Agent Assignment**: @market-research-agent (market-dynamics, growth-analysis, trend-identification)
- **Documentation Links**: [Market_Dynamics_Report.md](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Market_Dynamics_Report.md), [Growth_Drivers_Analysis.json](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Growth_Drivers_Analysis.json)
- **Steps**:
    1. Research market dynamics including growth drivers, market forces, cyclical patterns, and disruption factors (web_search)
    2. Document market dynamics assessment in Market_Dynamics_Report.md and create Growth_Drivers_Analysis.json (edit_file)
- **Success Criteria**:
    - Market dynamics data collected from multiple industry sources
    - Growth drivers, market forces, and disruption factors identified
    - Market_Dynamics_Report.md created with comprehensive dynamics analysis
    - Growth_Drivers_Analysis.json created with valid JSON structure
    - Report contains growth projections and key market drivers analysis
- **On Failure**: ESCALATE_TO_HUMAN (@market-research-lead) with dynamics analysis logs
- **Max Retries**: 2
- **Integration Points**: Market dynamics inform timing and strategic approach for market entry and competitive positioning.
- **Next Subtask**: P02-T02-S01 (Competitor Profiling)

## Rollback Procedures
1. Insufficient Market Data: Expand research scope or use alternative data sources
2. Competitive Intelligence Gaps: Conduct additional competitor research or primary interviews
3. Customer Insight Limitations: Supplement with primary customer research or surveys
4. Strategic Misalignment: Revisit research objectives and refocus analysis
5. Recommendation Uncertainty: Conduct scenario analysis and sensitivity testing

## Integration Points
- Industry structure analysis provides foundation for competitive positioning and market entry strategy development.
- Market dynamics inform timing and strategic approach for market entry and competitive positioning.

## Quality Gates
1. Research Depth: Comprehensive coverage of all market research dimensions
2. Data Reliability: High-quality sources and validated market data
3. Analysis Rigor: Systematic analysis with objective methodology
4. Strategic Relevance: Research findings directly support strategic decision-making
5. Actionability: Clear, implementable recommendations with specific next steps

## Success Criteria
- Comprehensive industry analysis with market structure and dynamics documented
- Detailed competitive intelligence with positioning opportunities identified
- Customer segmentation completed with detailed personas and journey maps
- Technology trends analysis with innovation roadmap implications
- Regulatory environment assessed with compliance requirements mapped
- Market gaps identified with strategic opportunities prioritized
- Strategic positioning framework developed with differentiation strategy
- Go-to-market strategy researched with channel and pricing recommendations
- Complete market research synthesis with validated insights
- Strategic recommendations developed for Phase 3 execution

## Risk Mitigation
- Data Quality Issues: Use multiple sources and cross-validate findings
- Market Volatility: Consider multiple scenarios and sensitivity analysis
- Competitive Intelligence: Use ethical research methods and public information
- Bias Prevention: Implement structured analysis frameworks and objective criteria
- Information Overload: Focus on strategic relevance and decision-critical insights

## Output Artifacts
- [Industry_Analysis_Report.md](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Industry_Analysis_Report.md)
- [Market_Structure_Map.json](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Market_Structure_Map.json)

## Next Action
Initiate comprehensive market research with @market-research-agent

## Post-Completion Action
Upon successful completion of all subtasks, update @Step.json and @DNA.json to reflect the task's SUCCEEDED status and any associated progress or outcomes. 