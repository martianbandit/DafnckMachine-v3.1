---
phase: P02
step: S03
task: T02
task_id: P02-S03-T02
title: Competitive Intelligence Deep Dive
previous_task: P02-S03-T01
next_task: P02-S03-T03
version: 3.1.0
source: Step.json
agent: "@market-research-agent"
orchestrator: "@uber-orchestrator-agent"
---

## Mission Statement
Conduct detailed competitive analysis including competitor profiling and positioning analysis to identify competitive advantages and market gaps.

## Description
Conduct detailed competitive analysis including competitor profiling and positioning analysis to identify competitive advantages and market gaps for DafnckMachine v3.1.

## Super-Prompt
"You are @market-research-agent responsible for conducting comprehensive market research for DafnckMachine v3.1. Your mission is to systematically analyze the industry landscape, competitive environment, customer segments, and market trends to provide strategic insights for product positioning and go-to-market strategy. Conduct thorough industry analysis, competitive intelligence gathering, customer segmentation, trend analysis, and strategic opportunity identification. Use both quantitative data and qualitative insights to create a comprehensive market understanding. Your research must be objective, data-driven, and provide actionable strategic recommendations for positioning, differentiation, and market entry. Document all findings in structured formats that support strategic decision-making and product development planning."

## MCP Tools Required
- web_search
- edit_file
- file_search
- list_dir

## Result We Want
- Detailed competitive intelligence with positioning opportunities
- Comprehensive competitor profiles and positioning analysis

## Add to Brain
- **Competitive Intelligence**: Competitor profiles, strategies, strengths, weaknesses, and market positioning

## Documentation & Templates
- [Competitive_Intelligence_Matrix.json](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Competitive_Intelligence_Matrix.json): Detailed competitor analysis and positioning
- [Competitor_Profiles.md](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Competitor_Profiles.md): Competitor profiles
- [Competitive_Positioning_Map.json](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Competitive_Positioning_Map.json): Positioning map
- [Feature_Comparison_Matrix.md](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Feature_Comparison_Matrix.md): Feature comparison

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
- Conduct detailed competitive analysis including competitor profiling and positioning analysis to identify competitive advantages and market gaps.

## Subtasks (Detailed)
### Subtask-01: Competitor Profiling
- **ID**: P02-T02-S01
- **Description**: Create detailed competitor profiles including business models, strategies, financial performance, market share, strengths and weaknesses.
- **Agent Assignment**: @market-research-agent (competitor-analysis, profiling, financial-analysis)
- **Documentation Links**: [Competitive_Intelligence_Matrix.json](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Competitive_Intelligence_Matrix.json), [Competitor_Profiles.md](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Competitor_Profiles.md)
- **Steps**:
    1. Research and analyze top competitors including business models, strategies, financial performance, and market positioning (web_search)
    2. Create detailed competitor profiles in Competitor_Profiles.md and populate Competitive_Intelligence_Matrix.json (edit_file)
- **Success Criteria**:
    - At least 5 key competitors identified and researched
    - Business models, strategies, and financial data collected for each competitor
    - Competitor_Profiles.md created with detailed competitor analysis
    - Competitive_Intelligence_Matrix.json created with valid JSON structure
    - Profiles include business models, strategies, strengths, weaknesses, and market share
- **On Failure**: ESCALATE_TO_HUMAN (@competitive-intelligence-lead) with profiling data
- **Max Retries**: 2
- **Integration Points**: Competitor profiles inform differentiation strategy and positioning framework development.
- **Next Subtask**: P02-T02-S02 (Competitive Positioning Analysis)

### Subtask-02: Competitive Positioning Analysis
- **ID**: P02-T02-S02
- **Description**: Analyze competitive positioning through market positioning maps, feature comparison, pricing analysis, and brand positioning assessment.
- **Agent Assignment**: @market-research-agent (positioning-analysis, competitive-mapping, feature-comparison)
- **Documentation Links**: [Competitive_Positioning_Map.json](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Competitive_Positioning_Map.json), [Feature_Comparison_Matrix.md](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Feature_Comparison_Matrix.md)
- **Steps**:
    1. Analyze competitive positioning including market positioning maps, feature comparison, and pricing analysis (web_search)
    2. Document positioning analysis in Feature_Comparison_Matrix.md and create Competitive_Positioning_Map.json (edit_file)
- **Success Criteria**:
    - Positioning maps created for key market dimensions
    - Feature comparison completed for all major competitors
    - Feature_Comparison_Matrix.md created with comprehensive feature analysis
    - Competitive_Positioning_Map.json created with valid JSON structure
    - Analysis identifies market gaps and positioning opportunities
- **On Failure**: ESCALATE_TO_HUMAN (@positioning-analyst) with positioning analysis data
- **Max Retries**: 2
- **Integration Points**: Positioning analysis reveals differentiation opportunities and informs strategic positioning framework.
- **Next Subtask**: P02-T03-S01 (Market Segmentation Analysis)

## Rollback Procedures
1. Insufficient Market Data: Expand research scope or use alternative data sources
2. Competitive Intelligence Gaps: Conduct additional competitor research or primary interviews
3. Customer Insight Limitations: Supplement with primary customer research or surveys
4. Strategic Misalignment: Revisit research objectives and refocus analysis
5. Recommendation Uncertainty: Conduct scenario analysis and sensitivity testing

## Integration Points
- Competitor profiles inform differentiation strategy and positioning framework development.
- Positioning analysis reveals differentiation opportunities and informs strategic positioning framework.

## Quality Gates
1. Research Depth: Comprehensive coverage of all market research dimensions
2. Data Reliability: High-quality sources and validated market data
3. Analysis Rigor: Systematic analysis with objective methodology
4. Strategic Relevance: Research findings directly support strategic decision-making
5. Actionability: Clear, implementable recommendations with specific next steps

## Success Criteria
- Detailed competitive intelligence with positioning opportunities identified
- Comprehensive competitor profiles and positioning analysis

## Risk Mitigation
- Data Quality Issues: Use multiple sources and cross-validate findings
- Market Volatility: Consider multiple scenarios and sensitivity analysis
- Competitive Intelligence: Use ethical research methods and public information
- Bias Prevention: Implement structured analysis frameworks and objective criteria
- Information Overload: Focus on strategic relevance and decision-critical insights

## Output Artifacts
- [Competitive_Intelligence_Matrix.json](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Competitive_Intelligence_Matrix.json)
- [Competitor_Profiles.md](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Competitor_Profiles.md)
- [Competitive_Positioning_Map.json](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Competitive_Positioning_Map.json)
- [Feature_Comparison_Matrix.md](mdc:01_Machine/04_Documentation/vision/Phase_2/03_Market_Research/Feature_Comparison_Matrix.md)

## Next Action
Initiate competitive intelligence research with @market-research-agent

## Post-Completion Action
Upon successful completion of all subtasks, update @Step.json and @DNA.json to reflect the task's SUCCEEDED status and any associated progress or outcomes. 