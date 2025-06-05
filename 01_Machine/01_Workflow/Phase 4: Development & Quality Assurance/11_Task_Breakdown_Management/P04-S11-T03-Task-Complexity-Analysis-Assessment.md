---
phase: P04
step: S11
task: T03
task_id: P04-S11-T03
title: Task Complexity Analysis & Assessment
previous_task: P04-S11-T02
next_task: P04-S11-T04
version: 3.1.0
source: Step.json
agent: "@task-planning-agent"
orchestrator: "@uber-orchestrator-agent"
---
## Output Artifacts Checklist
- [ ] 01_Machine/04_Documentation/vision/Phase_4_Development_QA/Task_Complexity_Analysis.md — Task_Complexity_Analysis.md: Task_Complexity_Analysis.md (missing)
- [ ] 01_Machine/04_Documentation/vision/Phase_4_Development_QA/Complexity_Assessment_Report.json — Complexity_Assessment_Report.json: Complexity_Assessment_Report.json (missing)
- [ ] 01_Machine/04_Documentation/vision/Phase_4_Development_QA/Task_Expansion_Strategy.md — Task_Expansion_Strategy.md: Task_Expansion_Strategy.md (missing)
- [ ] 01_Machine/04_Documentation/vision/Phase_4_Development_QA/Breakdown_Planning_Guide.json — Breakdown_Planning_Guide.json: Breakdown_Planning_Guide.json (missing)

## Mission Statement
Analyze the complexity of generated tasks to guide further breakdown and resource allocation for DafnckMachine v3.1.

## Description
Execute TaskMaster's complexity analysis to score tasks and get expansion recommendations. Develop a strategy for expanding tasks based on complexity analysis, prioritizing tasks for breakdown.

## Super-Prompt
"You are @task-planning-agent. Your mission is to analyze the complexity of all tasks, generate a complexity report, and develop a strategy for expanding and breaking down complex tasks."

## MCP Tools Required
- mcp_taskmaster-ai_analyze_project_complexity
- mcp_taskmaster-ai_complexity_report
- edit_file

## Result We Want
- Task complexity analysis is complete, and the report is generated.
- A clear task expansion strategy is documented.

## Add to Brain
- Resource Management: Efficient allocation strategies with timeline optimization and milestone tracking
- Progress Monitoring: Comprehensive analytics and reporting for development visibility and control

## Documentation & Templates
- [Task_Complexity_Analysis.md](mdc:01_Machine/04_Documentation/vision/Phase_4_Development_QA/Task_Complexity_Analysis.md)
- [Complexity_Assessment_Report.json](mdc:01_Machine/04_Documentation/vision/Phase_4_Development_QA/Complexity_Assessment_Report.json)
- [Task_Expansion_Strategy.md](mdc:01_Machine/04_Documentation/vision/Phase_4_Development_QA/Task_Expansion_Strategy.md)
- [Breakdown_Planning_Guide.json](mdc:01_Machine/04_Documentation/vision/Phase_4_Development_QA/Breakdown_Planning_Guide.json)

## Primary Responsible Agent
@task-planning-agent

## Supporting Agents
- None

## Agent Selection Criteria
@task-planning-agent is chosen for its expertise in complexity-analysis, task-assessment, expansion-strategy, and breakdown-planning.

## Tasks (Summary)
- Analyze project complexity for all tasks in tasks.json
- Review the complexity report to identify tasks needing expansion
- Formulate and document the expansion strategy

## Subtasks (Detailed)
### Subtask-01: Complexity Analysis Execution
- **ID:** P04-T03-S01
- **Description:** Execute TaskMaster's complexity analysis to score tasks and get expansion recommendations.
- **Agent:** @task-planning-agent
- **Documentation Links:**
  - [Task_Complexity_Analysis.md](mdc:01_Machine/04_Documentation/vision/Phase_4_Development_QA/Task_Complexity_Analysis.md)
  - [Complexity_Assessment_Report.json](mdc:01_Machine/04_Documentation/vision/Phase_4_Development_QA/Complexity_Assessment_Report.json)
- **Steps:**
    1. Analyze project complexity for all tasks in tasks.json using mcp_taskmaster-ai_analyze_project_complexity.
    2. Verify creation and content of scripts/task-complexity-report.json.
- **Success Criteria:**
    - File Exists: scripts/task-complexity-report.json
    - File Content Matches: task-complexity-report.json contains complexity data
- **On Failure:** NOTIFY_AND_CONTINUE (expansion will use defaults)
- **Max Retries:** 2
- **Integration Points:** The complexity report will guide the task expansion strategy.
- **Next Subtask:** P04-T03-S02

### Subtask-02: Expansion Strategy Development
- **ID:** P04-T03-S02
- **Description:** Develop a strategy for expanding tasks based on complexity analysis, prioritizing tasks for breakdown.
- **Agent:** @task-planning-agent
- **Documentation Links:**
  - [Task_Expansion_Strategy.md](mdc:01_Machine/04_Documentation/vision/Phase_4_Development_QA/Task_Expansion_Strategy.md)
  - [Breakdown_Planning_Guide.json](mdc:01_Machine/04_Documentation/vision/Phase_4_Development_QA/Breakdown_Planning_Guide.json)
- **Steps:**
    1. Review the complexity report to identify tasks needing expansion using mcp_taskmaster-ai_complexity_report.
    2. Formulate and document the expansion strategy based on the report using edit_file.
- **Success Criteria:**
    - Tool Output Contains: Complexity report displayed
    - File Exists: 01_Machine/04_Documentation/vision/Phase_4_Development_QA/Task_Expansion_Strategy.md
    - File Content Matches: Task_Expansion_Strategy.md contains the developed strategy
- **On Failure:** NOTIFY_AND_CONTINUE (default expansion strategy will be used)
- **Max Retries:** 1
- **Integration Points:** Guides the systematic breakdown of tasks in the next stage.
- **Next Subtask:** None

## Rollback Procedures
1. Task Structure Problems: Refine task breakdown and optimize dependency relationships

## Integration Points
- The complexity report will guide the task expansion strategy

## Quality Gates
- Task complexity analysis is complete and report generated
- Task expansion strategy is documented

## Success Criteria
- [ ] Task complexity analysis complete
- [ ] Task expansion strategy documented

## Risk Mitigation
- Task Breakdown Complexity: Systematic complexity analysis and iterative refinement processes

## Output Artifacts
- [Task_Complexity_Analysis.md](mdc:01_Machine/04_Documentation/vision/Phase_4_Development_QA/Task_Complexity_Analysis.md)
- [Complexity_Assessment_Report.json](mdc:01_Machine/04_Documentation/vision/Phase_4_Development_QA/Complexity_Assessment_Report.json)
- [Task_Expansion_Strategy.md](mdc:01_Machine/04_Documentation/vision/Phase_4_Development_QA/Task_Expansion_Strategy.md)
- [Breakdown_Planning_Guide.json](mdc:01_Machine/04_Documentation/vision/Phase_4_Development_QA/Breakdown_Planning_Guide.json)

## Next Action
Proceed to P04-S11-T04-Detailed-Task-Breakdown-Subtask-Generation.md

## Post-Completion Action
Upon completion, ensure the complexity report and expansion strategy are available for the next stage of detailed task breakdown and subtask generation. 