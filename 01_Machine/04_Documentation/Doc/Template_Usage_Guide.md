# Template Usage Guide for Workflow Step Documentation

## Overview
This guide explains how to use the Template-Step-Structure.md format for creating consistent, trackable, and agent-integrated workflow step documentation in DafnckMachine v3.1.

## Template Structure

### Required Sections

#### 1. Workflow Metadata
```markdown
## Workflow Metadata
- **Workflow-Step**: [Step Name]
- **TaskID**: [Unique Task ID]
- **Step ID**: [Sequential Number]
- **Version**: [Version Number]
- **LastUpdate**: [Date]
- **Previous Step**: [Previous Step File]
- **Next Step**: [Next Step File]
```

#### 2. Mission Statement
```markdown
## Mission Statement
[Clear, concise statement of the step's primary objective]
```

#### 3. Description
```markdown
## Description
[Detailed explanation of what this step accomplishes, including context and importance]
```

#### 4. Result We Want
```markdown
## Result We Want
[Specific, measurable outcomes expected from this step]
```

#### 5. Add to Brain
```markdown
## Add to Brain
- **Key Information Type**: Description of what gets stored
- **Another Information Type**: Additional knowledge captured
```

#### 6. Documentation & Templates
```markdown
## Documentation & Templates

### Required Documents
1. **Document_Name.md**: Purpose and description
2. **Another_Document.json**: Purpose and description

### Optional Documents
- **Optional_Document.md**: When this might be needed
```

#### 7. Super-Prompt
```markdown
## Super-Prompt
"[Comprehensive prompt for the assigned agent that includes context, specific instructions, and expected deliverables]"
```

#### 8. MCP Tools Required
```markdown
## MCP Tools Required
- **Tool Name**: Purpose and usage
- **Another Tool**: Purpose and usage
```

#### 9. Agent Selection & Assignment
```markdown
## Agent Selection & Assignment

### Primary Responsible Agent
**@agent-name** - `agent-slug`
- **Role**: [Agent's primary role]
- **Capabilities**: [Relevant capabilities for this step]
- **When to Use**: [Criteria for agent activation]

### Agent Selection Criteria
[Explanation of why this agent was chosen]

### Supporting Agents
1. **@supporting-agent**: [Role in this step]
2. **@another-agent**: [Role in this step]
```

#### 10. Task Breakdown (Hierarchical Structure)
```markdown
## Task Breakdown

### Phase Structure (Synchronized with DNA.json, Step.json, workflow_state.json)

```
[] Phase-01 (Strategic Level)
├── [] Task-01 (Tactical Level)
│   ├── [] Subtask-01 (Operational Level)
│   │   ├── [] 01 Agent Assignment: `@agent-name` - capability1, capability2 
│   │   ├── [] 02 Agent Prompt: "Detailed prompt for this specific subtask"
│   │   ├── [] 03 Documentation Links: Document1.md, Document2.md
│   │   ├── [] 04 MCP to use or context7 if needed 
│   │   ├── [] 05 Success Criteria: Specific completion criteria
│   │   └── [] 06 Integration Points: How this connects to other systems
│   ├── [] Subtask-02 (Operational Level)
│   │   ├── [] 01 Agent Assignment: `@agent-name` - capability3, capability4
│   │   ├── [] 02 Agent Prompt: "Detailed prompt for this specific subtask"
│   │   ├── [] 03 Documentation Links: Document3.md, Document4.md
│   │   ├── [] 04 MCP to use or context7 if needed 
│   │   ├── [] 05 Success Criteria: Specific completion criteria
│   │   └── [] 06 Integration Points: How this connects to other systems
```
```

#### 11. Success Criteria
```markdown
## Success Criteria
- [ ] Specific measurable outcome 1
- [ ] Specific measurable outcome 2
- [ ] All documentation completed and validated
- [ ] System ready for next workflow step
```

#### 12. Quality Gates
```markdown
## Quality Gates
1. **Gate Name**: Description and validation criteria
2. **Another Gate**: Description and validation criteria
```

#### 13. Risk Mitigation
```markdown
## Risk Mitigation
- **Risk Type**: Mitigation strategy
- **Another Risk**: Mitigation strategy
```

#### 14. Dependencies
```markdown
## Dependencies
### Input Dependencies
- Dependency from previous steps
- Required resources or information

### Output Dependencies
- What subsequent steps depend on
- Integration requirements
```

#### 15. Performance Metrics
```markdown
## Performance Metrics
- **Duration**: Expected time range
- **Quality Score**: Success rate or quality metrics
- **Completeness**: Coverage expectations
```

#### 16. Output Artifacts
```markdown
## Output Artifacts
1. **Artifact_Name.md**: Description and purpose
2. **Another_Artifact.json**: Description and purpose
```

#### 17. Rollback Procedures
```markdown
## Rollback Procedures
1. Step-by-step rollback instructions
2. Data recovery procedures
3. State restoration methods
```

#### 18. Integration Points
```markdown
## Integration Points
- **System/Tool Name**: Integration description
- **Another System**: Integration description
```

## JSON Synchronization

### DNA.json Integration
Ensure agent assignments in the documentation match the agent registry in DNA.json:
```json
{
  "agentRegistry": [
    {
      "agentName": "agent-name",
      "capabilities": ["capability1", "capability2"],
      "phases": ["phase-id"]
    }
  ]
}
```

### Step.json Integration
Update step status and metadata:
```json
{
  "currentStep": "step-id",
  "stepStatus": "in-progress",
  "lastUpdated": "timestamp"
}
```

### workflow_state.json Integration
Track hierarchical progress:
```json
{
  "currentPhase": "phase-id",
  "currentTask": "task-id",
  "currentSubtask": "subtask-id",
  "operational_item": "item-id",
  "subtasks_completed": 0,
  "operational_items_completed": 0
}
```

## Agent Assignment Guidelines

### Agent Discovery Process
1. **Review Agent Categories** in `01_Machine/02_Agents/`
2. **Check Agent Capabilities** in agent JSON files
3. **Validate Agent Interactions** for compatibility
4. **Verify Agent Permissions** for required access

### Agent Validation Checklist
- [ ] Agent's `roleDefinition` aligns with step objectives
- [ ] Agent's `whenToUse` criteria match current context
- [ ] Agent has required `groups` permissions
- [ ] Agent's `inputSpec` matches available inputs
- [ ] Agent's `outputSpec` produces required deliverables
- [ ] Agent's `interactsWith` includes necessary supporting agents

## Hierarchical Task Structure

### Level Definitions
- **Phase (Strategic Level)**: High-level workflow phase
- **Task (Tactical Level)**: Major deliverable or milestone
- **Subtask (Operational Level)**: Specific executable work item
- **Operational Items**: Granular tracking elements

### Checkbox System
- `[]` indicates trackable items
- Each level supports progress tracking
- Integration with workflow_state.json for automation

## Best Practices

### Documentation Quality
1. **Clarity**: Use clear, unambiguous language
2. **Completeness**: Include all required sections
3. **Consistency**: Follow template structure exactly
4. **Traceability**: Link to related documents and systems
5. **Maintainability**: Keep documentation up-to-date

### Agent Integration
1. **Specificity**: Provide detailed agent prompts
2. **Context**: Include relevant background information
3. **Validation**: Verify agent capabilities match requirements
4. **Coordination**: Define clear handoff points between agents

### System Integration
1. **JSON Sync**: Maintain consistency across configuration files
2. **State Tracking**: Support automated progress monitoring
3. **Dependencies**: Clearly define input/output relationships
4. **Rollback**: Provide comprehensive recovery procedures

## Common Patterns

### Multi-Agent Coordination
When multiple agents work together:
```markdown
### Supporting Agents
1. **@primary-agent**: Leads the overall process
2. **@specialist-agent**: Provides domain expertise
3. **@validation-agent**: Reviews and validates outputs
```

### Iterative Processes
For steps requiring iteration:
```markdown
### Iterative Workflow
1. Initial analysis and planning
2. Implementation and testing
3. Review and refinement
4. Validation and approval
```

### Quality Assurance
For steps requiring validation:
```markdown
### Quality Gates
1. **Completeness Check**: All deliverables present
2. **Quality Review**: Standards compliance verified
3. **Stakeholder Approval**: Formal sign-off obtained
```

## Troubleshooting

### Common Issues
1. **Agent Not Found**: Verify agent exists in registry
2. **Missing Dependencies**: Check previous step completion
3. **JSON Sync Errors**: Validate configuration file syntax
4. **Integration Failures**: Review system connection points

### Resolution Steps
1. Check agent registry for valid assignments
2. Validate JSON configuration files
3. Review dependency chains
4. Test integration points
5. Update documentation as needed

---

**Status**: Complete Template Guide  
**Last Updated**: 2025-01-27  
**Usage**: Reference for all workflow step documentation creation 