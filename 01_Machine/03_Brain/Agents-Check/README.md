# DafnckMachine v3.1 - Agent Validation & Repair Tools

## Overview

The Agents-Check directory contains comprehensive tools for validating, repairing, and managing agents in the DafnckMachine v3.1 ecosystem. The unified validators combine agent format validation, loading tests, system initialization, and automatic repair capabilities into powerful all-in-one tools.

## Available Tools

### 🐍 Python Version: `unified_agent_validator.py`
- **Recommended for**: Development, CI/CD integration, detailed analysis
- **Performance**: 67 agents in 0.298 seconds (97% success rate)
- **Features**: Advanced error handling, comprehensive reporting, memory efficient

### 🐚 Shell Version: `unified_agent_validator.sh`
- **Recommended for**: Quick validation, system administration, bash workflows
- **Dependencies**: Requires `jq` for JSON processing
- **Features**: Native bash implementation, detailed logging, robust error checking

## Quick Start

### Prerequisites

**For Python version:**
```bash
# Python 3.7+ required
python3 --version
```

**For Shell version:**
```bash
# Install jq if not available
# macOS
brew install jq

# Ubuntu/Debian
sudo apt-get install jq

# CentOS/RHEL
sudo yum install jq
```

### Basic Usage

**Validate all agents (Python):**
```bash
cd 01_Machine/03_Brain/Agents-Check
python3 unified_agent_validator.py --yes
```

**Validate all agents (Shell):**
```bash
cd 01_Machine/03_Brain/Agents-Check
./unified_agent_validator.sh -y
```

## Features

### 🔧 Repair Capabilities
- **Groups Format Repair**: Fixes complex `["edit", {...}]` format to simple `"edit"` strings
- **Broken Reference Repair**: Maps non-existent agent references to existing ones
- **Automatic Backup**: Creates timestamped backups before any modifications
- **Reference Mappings**: Built-in mappings for 50+ workflow agents

### 📋 Validation Features
- **JSON Syntax Validation**: Ensures proper JSON structure
- **Schema Compliance**: Validates against Agent-Format.json schema
- **Field Validation**: Checks required and optional fields
- **Loading Tests**: Simulates agent loading in chat mode
- **Connectivity Validation**: Verifies interactsWith references

### 🔄 System Integration
- **System Initialization**: Validates core configuration files
- **Directory Structure**: Checks required directories
- **Agent Registry**: Validates agent registry in DNA.json
- **.roomodes Sync**: Synchronizes valid agents to .roomodes file

## Command-Line Options

### Common Options (Both Versions)
```bash
-y, --yes           # Auto-yes mode (non-interactive)
-r, --report        # Generate detailed report
-a, --agents-only   # Test agents only, skip system init
-s, --system-only   # System initialization only
-h, --help          # Show help message
```

### Repair Options
```bash
--repair            # Run all repair operations before validation
--fix-groups        # Fix groups format issues only
--fix-references    # Fix broken interactsWith references only
--auto-repair       # Automatically repair issues during validation
--repair-only       # Run repairs only, skip validation
```

## Usage Examples

### 1. Interactive Mode
```bash
# Python version
python3 unified_agent_validator.py

# Shell version
./unified_agent_validator.sh
```

**Interactive Menu:**
```
--- Unified Agent Validator Menu ---
  [1] agent-file-1.json
  [2] agent-file-2.json
  ...
  [A] Test ALL agents
  [M] Test MULTIPLE agents (comma-separated)
  [T] Auto-yes mode (non-interactive)
  [S] System initialization only
  [R] Repair ALL agents
  [G] Fix groups format only
  [F] Fix broken references only
  [Q] Quit
```

### 2. Automated Validation
```bash
# Validate all agents automatically
python3 unified_agent_validator.py --yes

# Validate with detailed report
./unified_agent_validator.sh -y -r

# Agents only (skip system init)
python3 unified_agent_validator.py -a -y
```

### 3. Repair Operations
```bash
# Repair all issues then validate
python3 unified_agent_validator.py --repair

# Fix groups format only
./unified_agent_validator.sh --fix-groups

# Fix broken references only
python3 unified_agent_validator.py --fix-references

# Auto-repair during validation
./unified_agent_validator.sh --auto-repair

# Repair only (no validation)
python3 unified_agent_validator.py --repair-only
```

### 4. System Administration
```bash
# System initialization only
python3 unified_agent_validator.py --system-only

# Generate comprehensive report
./unified_agent_validator.sh --report

# Validate specific agents
python3 unified_agent_validator.py
# Then select [M] and enter: 1,3,5
```

## Output and Reports

### Console Output
Both tools provide color-coded console output:
- 🔍 **Blue**: Information messages
- ✅ **Green**: Success messages
- ⚠️ **Yellow**: Warnings
- ❌ **Red**: Errors
- 🔧 **Cyan**: Repair operations

### Generated Reports
Reports are saved to `unified_validation_report.md`:

```markdown
# Unified Agent Validation Report

## Agent Validation Summary
- Total agents tested: 67
- Valid agents: 65
- Failed agents: 2
- Success rate: 97.0%

## Repair Summary
- Repair mode: Active
- Groups format fixes: 3
- Reference fixes: 7
- Total repairs: 10

## System Status
- System Status: INITIALIZED
- Last Initialization: 2024-01-15T10:30:00
- Validation Errors: 0
- Validation Warnings: 2
```

## Backup System

Both tools automatically create backups before making any modifications:

**Backup Location:** `01_Machine/03_Brain/Agents-Check/backups/`

**Backup Format:** `{agent_name}_{YYYYMMDD_HHMMSS}.json`

**Example:**
```
backups/
├── uber-orchestrator-agent_20240115_103045.json
├── coding-agent_20240115_103046.json
└── test-orchestrator-agent_20240115_103047.json
```

## Reference Mappings

The tools include comprehensive reference mappings for workflow agents:

```bash
# Incomplete → Complete mappings
project-initiator → project-initiator-agent
uber-orchestrator → uber-orchestrator-agent
nlu-processor → nlu-processor-agent
elicitation → elicitation-agent
compliance-scope → compliance-scope-agent
# ... 50+ total mappings
```

**Invalid references automatically removed:**
- user, system, human, assistant
- Generic team references
- Non-existent agent names

## Troubleshooting

### Common Issues

**1. Permission Denied (Shell version)**
```bash
chmod +x unified_agent_validator.sh
```

**2. jq Command Not Found**
```bash
# Install jq (see Prerequisites section)
```

**3. Python Module Not Found**
```bash
# Ensure Python 3.7+ is installed
python3 --version
```

**4. Invalid JSON Errors**
```bash
# Use repair mode to fix common issues
./unified_agent_validator.sh --repair
```

**5. Path Issues**
```bash
# Always run from the Agents-Check directory
cd 01_Machine/03_Brain/Agents-Check
```

### Debug Mode

**Enable verbose logging (Python):**
```bash
python3 unified_agent_validator.py --yes 2>&1 | tee debug.log
```

**Enable verbose logging (Shell):**
```bash
./unified_agent_validator.sh -y 2>&1 | tee debug.log
```

### Log Files

**System logs:** `unified_system_init.log`
**Validation reports:** `unified_validation_report.md`

## Integration with CI/CD

### GitHub Actions Example
```yaml
- name: Validate Agents
  run: |
    cd 01_Machine/03_Brain/Agents-Check
    python3 unified_agent_validator.py --yes --auto-repair
```

### Pre-commit Hook
```bash
#!/bin/bash
cd 01_Machine/03_Brain/Agents-Check
./unified_agent_validator.sh --repair --yes
```

## Performance Benchmarks

| Tool | Agents | Time | Success Rate | Memory |
|------|--------|------|--------------|---------|
| Python | 67 | 0.298s | 97% | ~15MB |
| Shell | 67 | ~2.5s | 97% | ~5MB |

## Best Practices

### Development Workflow
1. **Daily validation**: Use `--auto-repair` for automatic issue fixing
2. **Pre-commit**: Use `--repair` for comprehensive validation
3. **Specific issues**: Use `--fix-groups` or `--fix-references` for targeted repairs

### Production Deployment
1. **Pre-deployment**: Run `--repair` to ensure all agents are valid
2. **System health**: Use `--system-only` for initialization checks
3. **Monitoring**: Use `--report` for detailed reporting
4. **Emergency repairs**: Use `--repair-only` for quick fixes

## File Structure

```
01_Machine/03_Brain/Agents-Check/
├── README.md                      # This file
├── unified_agent_validator.py     # Python version (recommended)
├── unified_agent_validator.sh     # Shell version
├── Agent-Format.json             # Validation schema
├── unified_validation_report.md   # Generated reports
├── unified_system_init.log       # System logs
├── backups/                      # Automatic backups
│   └── *.json                    # Timestamped backups
├── Agent-Health.py               # Legacy (use unified versions)
├── validate_agent_format.sh     # Legacy (use unified versions)
├── fix_groups_format.sh          # Legacy (integrated)
└── fix_broken_references.sh      # Legacy (integrated)
```

## Support

For issues or questions:
1. Check the troubleshooting section above
2. Review log files in `unified_system_init.log`
3. Run with `--help` for command-line options
4. Use `--report` for detailed validation results

## Version History

- **v3.1**: Unified validators with comprehensive repair capabilities
- **v3.0**: Separate validation and repair tools
- **v2.x**: Basic agent validation

---

**Recommendation**: Use the Python version (`unified_agent_validator.py`) for most use cases due to superior performance and error handling. Use the Shell version (`unified_agent_validator.sh`) for bash-heavy workflows or when Python is not available. 