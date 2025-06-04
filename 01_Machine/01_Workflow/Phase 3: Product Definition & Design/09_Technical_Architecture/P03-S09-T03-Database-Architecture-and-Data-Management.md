---
phase: P03
step: S09
task: T03
task_id: P03-S09-T03
title: Database Architecture and Data Management
previous_task: P03-S09-T02
next_task: P03-S09-T04
version: 3.1.0
source: Step.json
agent: "@system-architect-agent"
orchestrator: "@uber-orchestrator-agent"
---

# Mission Statement
Design the database architecture and establish a comprehensive data management strategy for DafnckMachine v3.1.

# Description
This task involves designing the database schema, relationship modeling, indexing strategy, normalization, query performance optimization, and establishing a holistic data management strategy covering storage, backup, migration, archiving, and governance.

# Super-Prompt
You are @system-architect-agent responsible for designing the database architecture and data management strategy for DafnckMachine v3.1. Your mission is to ensure efficient, secure, and scalable data storage, modeling, and management, supporting all application requirements and compliance needs.

# MCP Tools Required
- edit_file
- file_search
- web_search

# Result We Want
- Optimized database schema and comprehensive data management strategy

# Add to Brain
- Database Architecture: Schema design, relationship modeling, indexing, normalization, and query optimization
- Data Management: Storage, backup, migration, archiving, and governance

# Documentation & Templates
- [Database_Schema_Design.md](mdc:01_Machine/04_Documentation/Doc/Phase_3/09_Technical_Architecture/Database_Schema_Design.md)
- [Data_Model_Specifications.json](mdc:01_Machine/04_Documentation/Doc/Phase_3/09_Technical_Architecture/Data_Model_Specifications.json)
- [Data_Management_Strategy.md](mdc:01_Machine/04_Documentation/Doc/Phase_3/09_Technical_Architecture/Data_Management_Strategy.md)
- [Storage_Optimization_Plan.json](mdc:01_Machine/04_Documentation/Doc/Phase_3/09_Technical_Architecture/Storage_Optimization_Plan.json)

# Primary Responsible Agent
@system-architect-agent

# Supporting Agents
- @devops-agent
- @security-auditor-agent

# Agent Selection Criteria
The @system-architect-agent is chosen for its expertise in database design and data management, ensuring efficient, secure, and scalable solutions.

# Tasks (Summary)
- Design the database schema and establish a comprehensive data management strategy.

# Subtasks (Detailed)
## Subtask-01: Database Design & Schema Planning
- **ID**: P03-T03-S01
- **Description**: Design the database schema, including relationship modeling, indexing strategy, normalization, and query performance optimization.
- **Agent Assignment**: @system-architect-agent (database-design, schema-architecture)
- **Documentation Links**:
  - [Database_Schema_Design.md](mdc:01_Machine/04_Documentation/Doc/Phase_3/09_Technical_Architecture/Database_Schema_Design.md)
  - [Data_Model_Specifications.json](mdc:01_Machine/04_Documentation/Doc/Phase_3/09_Technical_Architecture/Data_Model_Specifications.json)
- **Steps**:
  1. Design database architecture: schema design, relationship modeling, indexing strategy, normalization optimization, query performance based on selected backend technology and application requirements (CognitiveProcessing)
  2. Document the database schema design, including diagrams, table definitions, relationships, and indexing strategies in Database_Schema_Design.md (edit_file)
  3. Populate Data_Model_Specifications.json with structured details of the data models and entities (edit_file)
- **Success Criteria**:
  - An optimized database schema ensuring efficient data modeling and query performance is documented in Database_Schema_Design.md and Data_Model_Specifications.json.
  - InternalState: Database schema and data model designed.
- **Integration Points**: The database design directly guides data management implementation, API development, and ORM configuration.
- **Next Subtask**: P03-T03-S02

## Subtask-02: Data Management Strategy
- **ID**: P03-T03-S02
- **Description**: Design a comprehensive data management strategy covering data storage, backup procedures, migration, archiving, and governance.
- **Agent Assignment**: @system-architect-agent (data-management, storage-strategy)
- **Documentation Links**:
  - [Data_Management_Strategy.md](mdc:01_Machine/04_Documentation/Doc/Phase_3/09_Technical_Architecture/Data_Management_Strategy.md)
  - [Storage_Optimization_Plan.json](mdc:01_Machine/04_Documentation/Doc/Phase_3/09_Technical_Architecture/Storage_Optimization_Plan.json)
- **Steps**:
  1. Design data management strategy: data storage solutions, backup and recovery procedures, data migration plans, data archiving policies, and data governance framework (CognitiveProcessing)
  2. Document the data management strategy in Data_Management_Strategy.md (edit_file)
  3. Outline the storage optimization plan in Storage_Optimization_Plan.json, if applicable (edit_file)
- **Success Criteria**:
  - A comprehensive data management strategy, including backup, migration, and governance procedures, is documented in Data_Management_Strategy.md.
  - InternalState: Data management strategy formulated.
- **Integration Points**: This strategy ensures data integrity, availability, security, and compliance throughout the application lifecycle.
- **Next Subtask**: None

# Rollback Procedures
- Redesign database schema and data management strategy based on feedback and technical constraints.

# Integration Points
- Database design and data management directly guide API development, data handling, and compliance.

# Quality Gates
- Optimized database schema and comprehensive data management strategy.

# Success Criteria
- Optimized database schema ensuring efficient data modeling and query performance.
- Comprehensive data management strategy, including backup, migration, and governance procedures.

# Risk Mitigation
- Comprehensive scalability testing and performance optimization strategies.
- Proactive security architecture and compliance validation.

# Output Artifacts
- [Database_Schema_Design.md](mdc:01_Machine/04_Documentation/vision/Phase_3/09_Technical_Architecture/Database_Schema_Design.md)
- [Data_Model_Specifications.json](mdc:01_Machine/04_Documentation/vision/Phase_3/09_Technical_Architecture/Data_Model_Specifications.json)
- [Data_Management_Strategy.md](mdc:01_Machine/04_Documentation/vision/Phase_3/09_Technical_Architecture/Data_Management_Strategy.md)
- [Storage_Optimization_Plan.json](mdc:01_Machine/04_Documentation/vision/Phase_3/09_Technical_Architecture/Storage_Optimization_Plan.json)

# Next Action
Initiate database architecture and data management strategy design with @system-architect-agent

# Post-Completion Action
Upon successful completion of all subtasks within this tactical task, ensure the @Step.json and @DNA.json files are updated to reflect its SUCCEEDED status and any associated progress or outcomes. 