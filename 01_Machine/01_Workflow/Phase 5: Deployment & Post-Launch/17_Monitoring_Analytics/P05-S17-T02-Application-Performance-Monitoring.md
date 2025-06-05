---
phase: P05
step: S17
task: T02
task_id: P05-S17-T02
title: Application Performance Monitoring
previous_task: P05-S17-T01
next_task: P05-S17-T03
version: 3.1.0
source: Step.json
agent: "@performance-load-tester-agent"
orchestrator: "@uber-orchestrator-agent"
---
## Output Artifacts Checklist
- [ ] 01_Machine/04_Documentation/vision/Phase_5/17_Monitoring_Analytics/APM_Implementation_Guide.md — APM_Implementation_Guide.md (missing)
- [ ] 01_Machine/04_Documentation/vision/Phase_5/17_Monitoring_Analytics/Performance_Metrics_Configuration.json — Performance_Metrics_Configuration.json (missing)
- [ ] 01_Machine/04_Documentation/vision/Phase_5/17_Monitoring_Analytics/Performance_Dashboard_Implementation.md — Performance_Dashboard_Implementation.md (missing)
- [ ] 01_Machine/04_Documentation/vision/Phase_5/17_Monitoring_Analytics/Metrics_Visualization.json — Metrics_Visualization.json (missing)

# Mission Statement
Implement and configure Application Performance Monitoring (APM) to track and analyze the performance of the application in real-time for DafnckMachine v3.1.

# Description
This task covers the implementation and configuration of APM agents, definition of key performance metrics, setup of transaction tracing, and enabling of error tracking. It also includes the creation of dashboards for visualizing performance metrics, KPIs, trend analysis, and capacity planning based on APM data.

# Super-Prompt
You are @performance-load-tester-agent. Your mission is to implement and configure robust APM for DafnckMachine v3.1, ensuring real-time performance tracking, transaction tracing, error monitoring, and actionable dashboard visualizations. Document all APM setup and dashboard procedures with clear guidelines and best practices.

# MCP Tools Required
- edit_file
- file_search
- list_dir
- run_terminal_cmd
- mcp_taskmaster-ai_get_task
- mcp_taskmaster-ai_set_task_status

# Result We Want
- APM agents deployed and configured
- Performance metrics, transaction tracing, and error tracking enabled
- Performance dashboards created and accessible

# Add to Brain
- APM agent setup
- Performance metrics definition
- Transaction tracing rules
- Error tracking parameters
- Dashboard structure and KPIs

# Documentation & Templates
- [APM_Implementation_Guide.md](mdc:01_Machine/04_Documentation/vision/Phase_5/17_Monitoring_Analytics/APM_Implementation_Guide.md)
- [Performance_Metrics_Configuration.json](mdc:01_Machine/04_Documentation/vision/Phase_5/17_Monitoring_Analytics/Performance_Metrics_Configuration.json)
- [Performance_Dashboard_Implementation.md](mdc:01_Machine/04_Documentation/vision/Phase_5/17_Monitoring_Analytics/Performance_Dashboard_Implementation.md)
- [Metrics_Visualization.json](mdc:01_Machine/04_Documentation/vision/Phase_5/17_Monitoring_Analytics/Metrics_Visualization.json)

# Primary Responsible Agent
@performance-load-tester-agent

# Supporting Agents
- @health-monitor-agent
- @data-analyst-agent
- @devops-agent

# Agent Selection Criteria
The @performance-load-tester-agent is chosen for its expertise in APM implementation, performance tracking, and dashboard visualization. It ensures comprehensive real-time monitoring and actionable insights for optimization.

# Tasks (Summary)
- Implement and configure APM agents
- Define performance metrics, transaction tracing, and error tracking
- Create and deploy performance dashboards

# Subtasks (Detailed)
## Subtask 1: APM Implementation & Configuration
- **ID**: P05-S17-T02-S01
- **Description**: Implement and configure APM agents, define key performance metrics, set up transaction tracing, and enable error tracking.
- **Agent**: @performance-load-tester-agent
- **Documentation Links**:
  - [APM_Implementation_Guide.md](mdc:01_Machine/04_Documentation/vision/Phase_5/17_Monitoring_Analytics/APM_Implementation_Guide.md)
  - [Performance_Metrics_Configuration.json](mdc:01_Machine/04_Documentation/vision/Phase_5/17_Monitoring_Analytics/Performance_Metrics_Configuration.json)
- **Steps**:
  1. Configure APM settings in Performance_Metrics_Configuration.json including agent setup, performance metrics, transaction tracing, and error tracking.
     - Tool: edit_file
     - Success Criteria:
       - File Exists: 01_Machine/04_Documentation/vision/Phase_5/17_Monitoring_Analytics/Performance_Metrics_Configuration.json
       - File Content Matches: Contains configurations for 'apmAgentSetup', 'performanceMetrics', 'transactionTracing', 'errorTracking'
  2. Deploy APM agents to the application environment as per APM_Implementation_Guide.md.
     - Tool: run_terminal_cmd
     - Success Criteria:
       - Exit Code: 0
       - Output Contains: APM agent deployment successful
       - HTTP Response: GET http://localhost/apm-agent-status returns HTTP 200 OK
- **Final Subtask Success Criteria**: APM is implemented with agents deployed, performance metrics configured, transaction tracing active, and error tracking enabled.
- **Integration Points**: APM data feeds into performance dashboards and provides crucial insights for optimization efforts.
- **Next Subtask**: P05-S17-T02-S02

## Subtask 2: Performance Metrics & Dashboards
- **ID**: P05-S17-T02-S02
- **Description**: Create dashboards for visualizing performance metrics, KPIs, trend analysis, and capacity planning based on APM data.
- **Agent**: @health-monitor-agent
- **Documentation Links**:
  - [Performance_Dashboard_Implementation.md](mdc:01_Machine/04_Documentation/vision/Phase_5/17_Monitoring_Analytics/Performance_Dashboard_Implementation.md)
  - [Metrics_Visualization.json](mdc:01_Machine/04_Documentation/vision/Phase_5/17_Monitoring_Analytics/Metrics_Visualization.json)
- **Steps**:
  1. Configure dashboard settings in Metrics_Visualization.json for performance KPIs, trend analysis, and capacity planning.
     - Tool: edit_file
     - Success Criteria:
       - File Exists: 01_Machine/04_Documentation/vision/Phase_5/17_Monitoring_Analytics/Metrics_Visualization.json
       - File Content Matches: Contains configurations for 'dashboardLayout', 'kpiDefinitions', 'trendAnalysisWidgets', 'capacityPlanningCharts'
  2. Deploy performance dashboards using the monitoring platform's tools as per Performance_Dashboard_Implementation.md.
     - Tool: run_terminal_cmd
     - Success Criteria:
       - Exit Code: 0
       - Output Contains: Performance dashboards deployed successfully
       - HTTP Response: GET http://monitoring-platform/dashboards/performance-dashboard-main returns HTTP 200 OK
- **Final Subtask Success Criteria**: Comprehensive performance dashboards are created and accessible, visualizing key metrics, trends, and capacity information.
- **Integration Points**: Dashboards provide actionable insights for proactive performance management and optimization decisions.
- **Next Subtask**: P05-S17-T03-S01

# Rollback Procedures
1. Debug APM agent deployment and restore monitoring functionality
2. Fix dashboard issues and restore visualization functionality

# Integration Points
- APM data feeds into dashboards and optimization workflows

# Quality Gates
- Real-time performance monitoring and alerting
- Actionable dashboard visualizations

# Success Criteria
- APM agents deployed and operational
- Dashboards created and accessible

# Risk Mitigation
- Performance bottleneck analysis
- Error tracking and alerting

# Output Artifacts
- [Performance_Metrics_Configuration.json](mdc:01_Machine/04_Documentation/vision/Phase_5/17_Monitoring_Analytics/Performance_Metrics_Configuration.json)
- [Metrics_Visualization.json](mdc:01_Machine/04_Documentation/vision/Phase_5/17_Monitoring_Analytics/Metrics_Visualization.json)

# Next Action
Implement and configure APM with @performance-load-tester-agent

# Post-Completion Action
Update @Step.json and @DNA.json to reflect SUCCEEDED status for this task and any associated progress or outcomes. 