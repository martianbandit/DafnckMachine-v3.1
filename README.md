$$$$$$$\   $$$$$$\  $$$$$$$$\ $$\   $$\  $$$$$$\  $$\   $$\ 
$$  __$$\ $$  __$$\ $$  _____|$$$\  $$ |$$  __$$\ $$ | $$  |
$$ |  $$ |$$ /  $$ |$$ |      $$$$\ $$ |$$ /  \__|$$ |$$  / 
$$ |  $$ |$$$$$$$$ |$$$$$\    $$ $$\$$ |$$ |      $$$$$  /  
$$ |  $$ |$$  __$$ |$$  __|   $$ \$$$$ |$$ |      $$  $$<   
$$ |  $$ |$$ |  $$ |$$ |      $$ |\$$$ |$$ |  $$\ $$ |\$$\  
$$$$$$$  |$$ |  $$ |$$ |      $$ | \$$ |\$$$$$$  |$$ | \$$\ 
\_______/ \__|  \__|\__|      \__|  \__| \______/ \__|  \__|


# DafnckMachine-V3.1
## Autonomous AI Workflow Orchestration Platform

### 🚀 **Quick Start** (30 seconds)

**For AI Agents**:
1. Start with `01_Machine/01_Workflow/{current_step}.md`
2. Read the **Agent Context** section at the top
3. Use `01_Machine/03_Brain/AGENT_INTERFACE.json` for lightweight configs
4. Output to `03_Project/{step_outputs}/`
5. Update progress and navigate to next step

**For Human Users**:
1. Review `02_Vision/Project_Structure_Integration.md` for architecture overview
2. Check `01_Machine/04_Documentation/01_System/Agent_Operations_Manual.md` for detailed guidance
3. Monitor progress in `01_Machine/01_Workflow/` files
4. Review outputs in `03_Project/` directories

---

## 🏗️ **System Architecture**

### **Three-Tier Design**
```
01_Machine/     # The Engine (How to execute)
├── 01_Workflow/    # Step-by-step execution plans with agent instructions
├── 02_Agents/      # Agent definitions and capabilities
├── 03_Brain/       # Intelligence system (DNA, STEP, GENESIS + lightweight interface)
└── 04_Documentation/ # System documentation and guides

02_Vision/      # The Strategy (What to build)
├── Project goals, vision, and strategic direction
├── High-level requirements and constraints
└── Success criteria and stakeholder expectations

03_Project/     # The Output (What gets built)
├── Actual project deliverables and implementation
├── Generated code, documentation, and artifacts
└── Structured outputs from each workflow step
```

### **Smart Brain System**
- **DNA.json**: Agent registry, capabilities, and communication protocols
- **STEP.JSON**: Execution engine with task mapping and state management
- **GENESIS.json**: Adaptive configuration and learning system
- **AGENT_INTERFACE.json**: ⚡ **Performance-optimized lightweight configs**

---

## ⚡ **Performance Optimizations**

### **For Maximum Performance**
1. **Use Lightweight Interface**: `AGENT_INTERFACE.json` instead of full Brain system
2. **Self-Contained Workflow Files**: All instructions embedded in workflow steps
3. **Caching System**: 5-minute cache for frequently accessed configurations
4. **Lazy Loading**: Only load what's needed for current step
5. **Parallel Task Execution**: Specified in performance configs

### **Agent Performance Features**
- **Step-specific configurations**: No need to load entire system
- **Embedded instructions**: Everything needed is in workflow files
- **Real-time state sync**: Progress tracked across all systems
- **Fallback mechanisms**: Graceful degradation for performance issues
- **Monitoring & alerts**: Automatic performance optimization

---

## 🤖 **Agent Integration**

### **Standard Agent Flow**
```bash
1. Load current workflow file (01_Workflow/{step}.md)
2. Read Agent Context section (immediate instructions)
3. Load step config from AGENT_INTERFACE.json
4. Reference 02_Vision/ for strategic alignment (if needed)
5. Execute numbered tasks (1.1, 1.2, 2.1, etc.)
6. Output results to 03_Project/{step_outputs}/
7. Update progress checklist in workflow file
8. Navigate to next step
```

### **Brain System Integration**
- **Lightweight Mode** (Recommended): Use `AGENT_INTERFACE.json`
- **Full Mode** (Complex operations): Access `DNA.json`, `STEP.JSON`, `GENESIS.json`
- **Hybrid Mode**: Start lightweight, escalate to full system if needed

---

## 📁 **Directory Navigation**

### **01_Machine/01_Workflow** - Your Primary Workspace
Each workflow file contains:
- **Agent Context**: Immediate instructions and configuration
- **Task Breakdown**: Numbered structure (1.1, 1.2, etc.)
- **Success Criteria**: Validation checklist
- **Navigation**: Links to previous/next steps
- **Brain Integration**: References to lightweight configs

### **02_Vision** - Strategic Reference
- **Project_Vision.md**: Core vision and mission
- **Strategic_Objectives.md**: Goals and success metrics
- **Project_Structure_Integration.md**: Architecture guide
- Use for strategic decisions and validation

### **03_Project** - Implementation Output
- **Structured directories** for each workflow step
- **Version-controlled artifacts** and deliverables
- **Integration-ready outputs** for next steps

---

## 🛠️ **Getting Started**

### **Step 1: System Initialization**
```bash
# Start with Project Initialization
cd "01_Machine/01_Workflow/Phase 0 : Project Setup"
open 00_Project_Initialization.md

# Agent will:
# 1. Read Agent Context section
# 2. Load config from AGENT_INTERFACE.json → "00_project_initialization"
# 3. Execute tasks 1.1 → 4.2
# 4. Output to 03_Project/initialization_results/
```

### **Step 2: User Requirements**
```bash
# Continue to User Briefing
cd "../Phase 1: Initial User Input & Project Inception"
open 01_User_Briefing.md

# Agent will:
# 1. Load briefing_agent config
# 2. Review previous step outputs
# 3. Conduct user interview (tasks 1.1 → 3.2)
# 4. Output to 03_Project/requirements/
```

### **Step 3: Discovery & Strategy**
```bash
# Proceed to Discovery
cd "../Phase 2: Discovery & Strategy"
open 02_Discovery_Strategy.md

# Agent will:
# 1. Load discovery_agent config
# 2. Reference 02_Vision/ for strategic alignment
# 3. Execute market research and analysis
# 4. Output to 03_Project/discovery/
```

---

## 📊 **Performance Monitoring**

### **Key Metrics**
- **Config Load Time**: < 2 seconds (target)
- **Agent Response Time**: < 30 seconds per task
- **Cache Hit Rate**: > 80%
- **Error Rate**: < 5%
- **Step Completion Time**: Varies by complexity

### **Performance Alerts**
- **Slow Config Load**: Auto-switch to cached version
- **High Error Rate**: Fallback to simplified mode
- **Memory Usage**: Automatic cleanup and optimization
- **State Sync Issues**: Real-time error correction

---

## 🔧 **Troubleshooting**

### **Common Issues**

**Agent can't find instructions**
→ Check Agent Context section in workflow file first

**Performance degradation**
→ Use AGENT_INTERFACE.json instead of full Brain system

**Output location unclear**
→ Follow 03_Project/ structure defined in workflow file

**Vision misalignment**
→ Validate against 02_Vision/Success_Criteria.md

**State synchronization problems**
→ Update workflow checklists and check Brain state management

### **Support Resources**
- **Agent Operations Manual**: `01_Machine/04_Documentation/01_System/Agent_Operations_Manual.md`
- **Integration Guide**: `02_Vision/Project_Structure_Integration.md`
- **Brain System Docs**: `01_Machine/03_Brain/` (DNA, STEP, GENESIS)
- **Lightweight Interface**: `01_Machine/03_Brain/AGENT_INTERFACE.json`

---

## 🎯 **Best Practices**

### **For Optimal Performance**
✅ **Start with workflow files** - All instructions are embedded  
✅ **Use lightweight configs** - AGENT_INTERFACE.json for most operations  
✅ **Follow numbered task structure** - 1.1, 1.2, 2.1, etc.  
✅ **Update progress in real-time** - Keep checklists current  
✅ **Reference vision strategically** - Only when needed for decisions  
✅ **Output to structured directories** - Follow 03_Project/ organization  

### **Avoid Performance Issues**
❌ **Don't load full Brain system** for simple tasks  
❌ **Don't skip Agent Context** sections in workflow files  
❌ **Don't ignore vision alignment** checks  
❌ **Don't create unstructured outputs**  
❌ **Don't forget progress updates**  

---

## 📈 **System Evolution**

### **Continuous Improvement**
- **GENESIS system** learns from agent performance
- **Automatic optimization** based on usage patterns
- **Real-time adaptation** to project requirements
- **Performance feedback loops** for system enhancement

### **Version History**
- **V3.1**: Performance optimization, lightweight agent interface
- **V3.0**: Smart Brain system (DNA, STEP, GENESIS)
- **V2.x**: Basic workflow orchestration
- **V1.x**: Initial agent framework

---

## 🚀 **Next Steps**

1. **Review** the Agent Operations Manual for detailed guidance
2. **Start** with 00_Project_Initialization.md for new projects
3. **Monitor** performance metrics and optimize as needed
4. **Contribute** feedback to improve system evolution

---

**🔄 Last Updated**: 2025-05-26  
**📞 Support**: Check documentation in 01_Machine/04_Documentation/  
**🧠 Brain Status**: Optimized for performance and usability  
**⚡ Performance**: Lightweight interface active

## Usage
- Use @agent-name to invoke a specific agent
- Agents can collaborate with each other as specified in their connectivity
- Each agent has specialized knowledge and capabilities
- **Note on Self-Collaboration:** In some agent definitions, an agent might be listed as collaborating with itself. This typically indicates an iterative internal process or a recursive capability within that agent.

## Available Agents 

## @development-orchestrator-agent

**🛠️ Development Orchestrator Agent**

### Collaborates with:
- @coding-agent
- @code-reviewer-agent
- @test-orchestrator-agent
- @devops-agent
- @prd-architect-agent
- @system-architect-agent
- @task-planning-agent

### Responsibilities:
- Orchestrates the overall development process, ensuring smooth transitions between coding, review, testing, and deployment.
- Manages the flow of tasks and coordinates between different development-focused agents.
- **Assists with Git workflow automation when Taskmaster subtasks are marked 'done', including prompting for commits, pre-filling commit messages based on task details, and guiding the Changeset process if applicable (referencing `dev_workflow.mdc` and `changeset.mdc`).**

## @prd-architect-agent

**📝 PRD Architect Agent**

### Collaborates with:
- @project-initiator-agent
- @market-research-agent
- @system-architect-agent
- @ux-researcher-agent
- @prd-architect-agent
- @documentation-agent

### Responsibilities:
- Leads the creation and maintenance of the Product Requirements Document (PRD).
- Translates user needs, market research, and technical specifications into a comprehensive PRD.
- Ensures the PRD is clear, actionable, and aligned with project goals.
- **If a PRD is not provided by the user, this agent actively facilitates discussions to gather requirements, then leverages the `scripts/example_prd.txt` (referenced in `taskmaster.mdc`) as a template to draft an initial PRD for user review and subsequent parsing by Taskmaster.**

## @uber-orchestrator-agent

**🎩 Uber Orchestrator Agent (Talk with me)**

### Collaborates with:
- @development-orchestrator-agent
- @marketing-strategy-orchestrator
- @test-orchestrator-agent
- @swarm-scaler-agent
- @health-monitor-agent
- @devops-agent
- @system-architect-agent
- @security-auditor-agent

### Responsibilities:
- Acts as the central coordinator for all other orchestrator and specialized agents.
- Oversees the entire project lifecycle, ensuring all phases and agent collaborations proceed smoothly.
- Manages high-level project strategy and delegates tasks to appropriate sub-orchestrators.
- **Explicitly manages and orchestrates the project's workflow state by: ensuring `01_Machine/03_Brain/Step.json` is updated to reflect current progress; and driving the automatic transitions between phases, steps, and tasks as dictated by `01_Machine/03_Brain/DNA.json` and the `cursor_rules.mdc` directive.**

## @devops-agent

**⚙️ DevOps Agent**

### Collaborates with:
- @coding-agent
- @security-auditor-agent
- @performance-load-tester-agent
- @health-monitor-agent
- @system-architect-agent
- @system-architect-agent
- @test-orchestrator-agent

### Responsibilities:
- Manages Continuous Integration/Continuous Deployment (CI/CD) pipelines.
- Oversees infrastructure, environment setup, and deployment strategies (collaborating with @adaptive-deployment-strategist-agent).
- Monitors application health and performance (collaborating with @health-monitor-agent).
- Ensures smooth and efficient build, test, and release processes.
- **Performs pre-flight configuration checks before AI-dependent Taskmaster tools are invoked. This includes verifying that the necessary API keys for the currently configured AI models (as specified in `.taskmasterconfig`) are present and accessible in the appropriate location (e.g., `.cursor/mcp.json`), and issuing warnings if configurations are incomplete, as detailed in `taskmaster.mdc`.**

## @code-reviewer-agent

**🧐 Code Reviewer Agent**

### Collaborates with:
- @coding-agent
- @security-auditor-agent
- @performance-load-tester-agent
- @devops-agent
- @system-architect-agent
- @test-orchestrator-agent

### Responsibilities:
- Conducts thorough code reviews to ensure code quality, adherence to standards, and best practices.
- Checks for potential bugs, security vulnerabilities, and performance issues.
- Provides constructive feedback to the @coding-agent.
- Verifies that code changes align with the technical specifications and PRD.
- **During code reviews, actively identifies new or recurring code patterns, anti-patterns, or deviations from best practices that are not yet covered by existing rules (as outlined in `self_improve.mdc` and `.cursor/rules/`). It may then suggest, flag, or draft new/updated rules for consideration to enhance the project's automated checks and overall code quality.**