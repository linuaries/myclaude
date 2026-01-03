# Claude Code Multi-Agent Workflow System - Complete Documentation

> AI-powered development automation with multi-backend execution (Codex/Claude/Gemini)

## Table of Contents

1. [API Reference](#api-reference)
2. [Code Overview](#code-overview)
3. [User Manual](#user-manual)
4. [Developer Handbook](#developer-handbook)
5. [Appendices](#appendices)

---

# API Reference

## codeagent-wrapper CLI

### Command Syntax

```bash
codeagent-wrapper [OPTIONS] [TASK] [WORKDIR]
codeagent-wrapper - <<'EOF'
<TASK_CONTENT>
EOF
codeagent-wrapper --parallel <<'EOF'
---TASK---
id: <unique_id>
workdir: /path
dependencies: <dep1>,<dep2>
---CONTENT---
<TASK>
EOF
```

### Options

| Option | Description |
|--------|-------------|
| `--backend <codex\|claude\|gemini>` | Select AI backend (default: codex) |
| `--parallel` | Enable parallel task execution |
| `--full-output` | Show complete task messages (debug mode) |
| `--skip-permissions` | Skip Claude CLI permission checks |
| `-h, --help` | Show help message |

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `CODEX_TIMEOUT` | 7200000 | Timeout in milliseconds (2 hours) |
| `CODEAGENT_SKIP_PERMISSIONS` | false | Skip Claude permission checks |
| `CODEAGENT_MAX_PARALLEL_WORKERS` | unlimited | Max concurrent tasks |
| `BMAD_REVIEW_MODE` | standard | Review mode (standard/enhanced) |

### Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | General error (missing args, no output) |
| 124 | Timeout |
| 127 | Backend command not found |
| 130 | Interrupted (Ctrl+C) |

## Slash Commands API

### Workflow Commands

| Command | Description | Module |
|---------|-------------|--------|
| `/dev` | Minimal dev workflow with concurrent execution | dev |
| `/bmad-pilot` | Full BMAD agile workflow | bmad |
| `/requirements-pilot` | Requirements-driven development | requirements |

### Development Commands

| Command | Description | Agent |
|---------|-------------|-------|
| `/code` | Direct implementation | code |
| `/debug` | Systematic debugging | debug |
| `/test` | Testing strategy | develop |
| `/optimize` | Performance tuning | develop |
| `/bugfix` | Bug resolution | bugfix |
| `/refactor` | Code improvement | develop |
| `/review` | Code validation | review |
| `/ask` | Technical consultation | consultant |
| `/docs` | Documentation | docs |
| `/think` | Advanced analysis | gpt5 |

## Skills API

### codeagent Skill

```bash
codeagent-wrapper --backend <backend> - <<'EOF'
<TASK>
EOF
```

Supports:
- File references: `@file` or `@path/to/file`
- Working directory: `- [workdir]`
- Session resume: `resume <session_id>`
- Parallel execution: `--parallel` with `---TASK---` blocks

### codex Skill

```bash
codex-wrapper - [workdir] <<'EOF'
<TASK>
EOF
```

### product-requirements Skill

Used by BMAD workflow for PRD generation.

### prototype-prompt-generator Skill

Generates UI/UX prompts for design systems.

---

# Code Overview

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Claude Code (Orchestrator)                │
│                    (Planning, Context, Verification)         │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                   codeagent-wrapper                          │
│              (Multi-backend abstraction layer)               │
├─────────────┬─────────────┬──────────────────────────────────┤
│   Codex     │   Claude    │   Gemini                        │
│   Backend   │   Backend   │   Backend                       │
│  (default)  │             │                                 │
└─────────────┴─────────────┴──────────────────────────────────┘
```

## Core Components

### 1. Installation System (`install.py`)

**Purpose**: JSON-driven modular installer

**Key Functions**:
- `parse_args()` - CLI argument parsing
- `load_config()` - Load and validate config.json
- `execute_module()` - Execute module operations
- `op_copy_dir()`, `op_copy_file()`, `op_merge_dir()` - File operations
- `op_run_command()` - Shell command execution
- `rollback()` - Installation rollback on failure

**Supported Operations**:
- `copy_dir` - Copy entire directory
- `copy_file` - Copy single file
- `merge_dir` - Merge subdirs into install dir
- `merge_json` - Deep merge JSON files
- `run_command` - Execute shell command

### 2. Configuration System

**config.json** - Module definitions and operations

```json
{
  "version": "1.0",
  "install_dir": "~/.claude",
  "modules": {
    "dev": { "enabled": true, "operations": [...] },
    "bmad": { "enabled": false, "operations": [...] },
    "requirements": { "enabled": false, "operations": [...] },
    "essentials": { "enabled": true, "operations": [...] }
  }
}
```

**config.schema.json** - JSON Schema validation

**hooks-config.json** - Hook event bindings

### 3. Workflow Systems

#### Dev Workflow (`dev-workflow/`)

**Flow**: Backend Selection → Requirements → Analysis → Plan → Concurrent Dev → Testing

**Key Files**:
- `commands/dev.md` - Workflow orchestrator
- `agents/dev-plan-generator.md` - Plan document generator

#### BMAD Workflow (`bmad-agile-workflow/`)

**Agents**:
| Agent | File | Role |
|-------|------|------|
| bmad-po | bmad-po.md | Product Owner |
| bmad-architect | bmad-architect.md | System Architect |
| bmad-sm | bmad-sm.md | Scrum Master |
| bmad-dev | bmad-dev.md | Developer |
| bmad-review | bmad-review.md | Code Reviewer |
| bmad-qa | bmad-qa.md | QA Engineer |
| bmad-orchestrator | bmad-orchestrator.md | Main orchestrator |

#### Requirements Workflow (`requirements-driven-workflow/`)

**Agents**:
| Agent | File | Role |
|-------|------|------|
| requirements-generate | requirements-generate.md | Requirements generation |
| requirements-code | requirements-code.md | Code implementation |
| requirements-review | requirements-review.md | Code review |
| requirements-testing | requirements-testing.md | Testing strategy |

#### Development Essentials (`development-essentials/`)

**Commands**: code, debug, test, optimize, bugfix, refactor, review, ask, docs, think

**Agents**: code, bugfix, debug, develop

### 4. Skills System (`skills/`)

| Skill | Purpose |
|-------|---------|
| codeagent | Multi-backend execution wrapper |
| codex | Codex CLI integration |
| product-requirements | PRD generation |
| prototype-prompt-generator | UI/UX prompt generation |
| gemini | Gemini CLI integration |

### 5. Hooks System (`hooks/`)

**Event Types**:
- `UserPromptSubmit` - After user submits prompt
- `PostToolUse` - After tool execution
- `Stop` - Session ends

**Built-in Hooks**:
- `skill-activation-prompt.sh` - Auto-suggest skills
- `pre-commit.sh` - Code quality checks

## Directory Structure

```
myclaude/
├── install.py                    # Modular installer
├── config.json                   # Module configuration
├── config.schema.json            # JSON Schema validation
├── README.md                     # Project overview
├── README_CN.md                 # Chinese overview
├── CHANGELOG.md                 # Version history
│
├── dev-workflow/                # Minimal dev workflow
│   ├── commands/dev.md
│   └── agents/dev-plan-generator.md
│
├── bmad-agile-workflow/         # Full agile methodology
│   ├── commands/bmad-pilot.md
│   ├── agents/
│   │   ├── bmad-po.md           # Product Owner
│   │   ├── bmad-architect.md    # Architect
│   │   ├── bmad-sm.md           # Scrum Master
│   │   ├── bmad-dev.md          # Developer
│   │   ├── bmad-review.md       # Reviewer
│   │   ├── bmad-qa.md           # QA Engineer
│   │   └── bmad-orchestrator.md # Orchestrator
│   └── .claude-plugin/marketplace.json
│
├── requirements-driven-workflow/ # Lightweight workflow
│   ├── commands/requirements-pilot.md
│   └── agents/
│       ├── requirements-generate.md
│       ├── requirements-code.md
│       ├── requirements-review.md
│       └── requirements-testing.md
│
├── development-essentials/      # Daily commands
│   ├── commands/
│   │   ├── code.md, debug.md, test.md
│   │   ├── optimize.md, bugfix.md
│   │   ├── refactor.md, review.md
│   │   ├── ask.md, docs.md, think.md
│   └── agents/
│       ├── code.md, bugfix.md, debug.md
│       └── develop.md
│
├── skills/                      # Claude Code skills
│   ├── codeagent/SKILL.md       # Multi-backend execution
│   ├── codex/SKILL.md           # Codex integration
│   ├── product-requirements/SKILL.md
│   ├── prototype-prompt-generator/SKILL.md
│   └── skill-rules.json         # Auto-suggestion rules
│
├── docs/                        # Documentation
│   ├── CODEAGENT-WRAPPER.md     # Wrapper reference
│   ├── BMAD-WORKFLOW.md         # Agile workflow guide
│   ├── REQUIREMENTS-WORKFLOW.md # Requirements guide
│   ├── DEVELOPMENT-COMMANDS.md  # Commands reference
│   ├── HOOKS.md                 # Hooks guide
│   ├── QUICK-START.md           # Getting started
│   └── PLUGIN-SYSTEM.md         # Plugin management
│
├── hooks/                       # Hook scripts
│   ├── hooks-config.json        # Hook configuration
│   ├── skill-activation-prompt.sh
│   └── pre-commit.sh
│
├── memorys/                     # Core instructions
│   └── CLAUDE.md                # Role definition
│
├── .claude-plugin/              # Plugin metadata
│   └── marketplace.json
│
└── output-styles/               # Output formatting
    └── bmad-phase-context.md
```

---

# User Manual

## Installation

### Prerequisites

- Python 3.8+
- Claude Code At least one backend: CLI installed
- Codex CLI, Claude CLI, or Gemini CLI

### Installation Methods

#### Method 1: Plugin Command (Recommended)

```bash
/plugin marketplace add cexll/myclaude
```

#### Method 2: Python Installer

```bash
git clone https://github.com/cexll/myclaude.git
cd myclaude
python3 install.py --install-dir ~/.claude
```

#### Method 3: Selective Installation

```bash
# Install specific module
python3 install.py --module dev

# List available modules
python3 install.py --list-modules

# Install multiple modules
python3 install.py --module bmad,requirements,essentials
```

### Verify Installation

```bash
# Check installed modules
cat ~/.claude/installed_modules.json

# Test codeagent-wrapper
codeagent-wrapper "test task"
```

## Quick Start

### Your First Workflow

```bash
# Simple feature with direct command
/code "Add input validation for email fields"

# Debug an issue
/debug "API returns 500 on missing parameters"

# Add tests
/test "Create unit tests for validation logic"
```

### Using BMAD Workflow

```bash
/bmad-pilot "Build a todo list API with user authentication"
```

**What happens**:
1. Repository context scan
2. Product Owner generates PRD (score ≥ 90 required)
3. Architect designs system (score ≥ 90 required)
4. Scrum Master creates sprint plan
5. Developer implements code
6. Reviewer validates code
7. QA runs tests

### Using Requirements Workflow

```bash
/requirements-pilot "Add pagination to user list endpoint"
```

**What happens**:
1. Generate functional requirements (score ≥ 90)
2. Implement code
3. Review implementation
4. Create tests

### Using Dev Workflow

```bash
/dev "Implement user login feature"
```

**What happens**:
1. Backend selection (codex/claude/gemini)
2. Requirements clarification
3. Code analysis and task typing
4. Generate dev-plan.md
5. Concurrent development (2-5 tasks)
6. Testing (≥90% coverage)

## Workflow Selection Guide

| Scenario | Command | Best For |
|----------|---------|----------|
| Complex feature with architecture | `/bmad-pilot` | Enterprise projects, multi-sprint features |
| Clear requirements, fast iteration | `/requirements-pilot` | Simple features, prototypes |
| Well-defined task, no overhead | `/code` | Quick implementations, config changes |
| Bug investigation | `/debug` | Complex debugging, root cause analysis |
| Bug fix | `/bugfix` | Specific, well-scoped bugs |
| Code review | `/review` | Quality, security, best practices |
| Performance optimization | `/optimize` | Slow code, resource usage |
| Code improvement | `/refactor` | Restructuring without behavior change |
| Technical guidance | `/ask` | Design patterns, architecture advice |
| Documentation | `/docs` | API docs, README, code comments |
| Complex analysis | `/think` | Architectural decisions, strategic planning |

## Backend Selection

### When to Use Each Backend

| Backend | Strengths | Best For |
|---------|-----------|----------|
| **Codex** | Deep code understanding, complex refactoring | Backend logic, algorithms, large refactors |
| **Claude** | Simple tasks, reasoning, documentation | Quick fixes, docs, prompts |
| **Gemini** | UI/UX prototyping | Frontend components, styling, layouts |

### Using Backends

```bash
# Default (Codex)
codeagent-wrapper "implement feature"

# Explicit backend selection
codeagent-wrapper --backend claude "quick fix"
codeagent-wrapper --backend gemini "create React component"

# In /dev workflow, select at start
/dev "feature description"
# Q: Which backends are allowed?
# A: codex, claude, gemini
```

## Parallel Execution

```bash
codeagent-wrapper --parallel <<'EOF'
---TASK---
id: backend_1701234567
workdir: /project/backend
---CONTENT---
implement /api/users endpoints

---TASK---
id: frontend_1701234568
workdir: /project/frontend
---CONTENT---
build Users page

---TASK---
id: tests_1701234569
dependencies: backend_1701234567, frontend_1701234568
---CONTENT---
add integration tests
EOF
```

## Session Management

```bash
# First task (session starts)
codeagent-wrapper "add login feature"
# Output: SESSION_ID: 019a7247-ac9d-71f3-89e2-a823dbd8fd14

# Resume session
codeagent-wrapper resume 019a7247-ac9d-71f3-89e2-a823dbd8fd14 - <<'EOF'
now add password reset functionality
EOF
```

## File References

```bash
# Single file
codeagent-wrapper "analyze @src/auth.ts"

# Multiple files
codeagent-wrapper "refactor @src/auth.ts and @src/middleware.ts"

# Directory
codeagent-wrapper "find security issues in @src"
```

---

# Developer Handbook

## Development Setup

### Clone and Setup

```bash
git clone https://github.com/cexll/myclaude.git
cd myclaude

# Create virtual environment
python3 -m venv venv
source venv/bin/activate

# Install development dependencies
pip install -e .

# Run tests
python -m pytest
```

### Building codeagent-wrapper

```bash
cd codeagent-wrapper
go build -o ~/.claude/bin/codeagent-wrapper
```

### Running Installer

```bash
# Standard install
python3 install.py --install-dir ~/.claude

# Verbose mode
python3 install.py --install-dir ~/.claude --verbose

# Force overwrite
python3 install.py --install-dir ~/.claude --force

# List modules
python3 install.py --list-modules
```

## Adding New Modules

### 1. Create Module Directory

```
new-module/
├── commands/           # Slash command definitions
│   └── new-command.md
├── agents/             # Agent definitions
│   └── new-agent.md
├── docs/
│   └── NEW-MODULE.md   # Module documentation
└── .claude-plugin/
    └── marketplace.json
```

### 2. Define in config.json

```json
"new-module": {
  "enabled": false,
  "description": "Description of module",
  "operations": [
    {
      "type": "merge_dir",
      "source": "new-module",
      "description": "Merge commands and agents"
    },
    {
      "type": "copy_file",
      "source": "docs/NEW-MODULE.md",
      "target": "docs/NEW-MODULE.md"
    }
  ]
}
```

### 3. Register in marketplace.json

```json
{
  "name": "new-module",
  "displayName": "New Module",
  "description": "What this module does",
  "version": "1.0.0",
  "author": "Your Name",
  "category": "workflow",
  "keywords": ["keyword1", "keyword2"],
  "commands": ["new-command"],
  "agents": ["new-agent"]
}
```

## Creating New Commands

### Command File Structure (`commands/*.md`)

```markdown
# /command-name - Brief Description

## Overview

What this command does.

## Usage

```bash
/command-name "task description"
```

## Process

1. Step 1
2. Step 2
3. Step 3

## Examples

```bash
/command-name "Do something specific"
```

## When to Use

- Use case 1
- Use case 2

## Related Commands

- `/other-command` - Related functionality
```

### Agent File Structure (`agents/*.md`)

```markdown
# agent-name

## Role

Brief role description.

## Responsibilities

- Responsibility 1
- Responsibility 2

## Input

- What the agent receives

## Output

- What the agent produces

## Quality Criteria

- Criterion 1
- Criterion 2
```

## Creating New Skills

### Skill File Structure (`skills/name/SKILL.md`)

```markdown
---
name: skill-name
description: Brief skill description
---

# Skill Name

## Overview

What the skill does.

## When to Use

- Use case 1
- Use case 2

## Usage

```bash
skill-command "task description"
```

## Parameters

- `param` (required/optional): Description

## Examples

```bash
# Example 1
skill-command "do something"
```

## Return Format

```
Output format description
```

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `VAR_NAME` | default | Description |
```

## Hooks Development

### Creating Custom Hooks

```bash
#!/bin/bash
# hooks/my-hook.sh

set -e  # Exit on error
set -x  # Print commands (debug)

# Hook logic here
echo "Running custom hook"

# Access environment
PROJECT_DIR="$CLAUDE_PROJECT_DIR"
```

### Registering Hooks

Edit `~/.claude/settings.json`:

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "$CLAUDE_PROJECT_DIR/hooks/my-hook.sh"
          }
        ]
      }
    ]
  }
}
```

### Hook Event Types

| Event | When | Use Cases |
|-------|------|-----------|
| `UserPromptSubmit` | After user submits | Auto-suggest skills, inject context |
| `PostToolUse` | After tool execution | Validate output, linting |
| `Stop` | Session ends | Cleanup, commit, reports |

## Testing

### Running Tests

```bash
# All tests
python -m pytest

# Specific module
python -m pytest tests/dev-workflow/

# With coverage
python -m pytest --cov=. --cov-report=html
```

### Writing Tests

```python
# tests/test_module.py
import pytest
from install import load_config, parse_args

def test_parse_args():
    args = parse_args(["--install-dir", "/test"])
    assert args.install_dir == "/test"

def test_load_config():
    config = load_config("config.json")
    assert "modules" in config
```

## Coding Standards

### Python (install.py)

- Type hints for all functions
- Docstrings for public functions
- Max line length: 100 characters
- Follow PEP 8

### Shell Scripts (hooks)

- `set -e` for error handling
- `set -u` for undefined variables
- Use `#!/bin/bash`
- Comment complex logic

### Markdown (docs, commands)

- Use ATX headings (`#`, `##`)
- Code blocks with language
- Tables for structured data
- Max line length: 100 characters

## Contribution Workflow

1. **Fork** the repository
2. **Create** a feature branch: `git checkout -b feature/new-module`
3. **Make** changes following coding standards
4. **Test** your changes: `python -m pytest`
5. **Commit**: `git commit -m "feat: Add new module"`
6. **Push**: `git push origin feature/new-module`
7. **Create** Pull Request

## Versioning

This project follows [Semantic Versioning](https://semver.org/):

- **MAJOR**: Breaking changes
- **MINOR**: New features (backward compatible)
- **PATCH**: Bug fixes

Changelog maintained in `CHANGELOG.md`.

---

# Appendices

## Appendix A: Configuration Reference

### config.json Full Schema

```json
{
  "version": "string",
  "install_dir": "string",
  "log_file": "string",
  "modules": {
    "module-name": {
      "enabled": boolean,
      "description": "string",
      "operations": [
        {
          "type": "copy_dir|copy_file|merge_dir|merge_json|run_command",
          "source": "string",
          "target": "string",
          "description": "string",
          "env": { "VAR": "value" }
        }
      ]
    }
  }
}
```

### hooks-config.json

```json
{
  "UserPromptSubmit": [
    {
      "hooks": [
        {
          "type": "command",
          "command": "$CLAUDE_PROJECT_DIR/hooks/script.sh"
        }
      ]
    }
  ]
}
```

### skill-rules.json

```json
{
  "skills": {
    "skill-name": {
      "type": "execution|domain",
      "enforcement": "suggest|require",
      "priority": "high|medium|low",
      "promptTriggers": {
        "keywords": ["keyword1", "keyword2"],
        "intentPatterns": ["regex pattern"]
      }
    }
  }
}
```

## Appendix B: Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `INSTALL_DIR` | `~/.claude` | Installation directory |
| `CODEX_TIMEOUT` | 7200000 | Codex timeout (ms) |
| `CODEAGENT_SKIP_PERMISSIONS` | false | Skip Claude permissions |
| `CODEAGENT_MAX_PARALLEL_WORKERS` | unlimited | Max parallel tasks |
| `BMAD_REVIEW_MODE` | standard | Review depth |

## Appendix C: File Paths After Installation

```
~/.claude/
├── bin/
│   └── codeagent-wrapper    # Main executable
├── CLAUDE.md                # Core instructions
├── commands/                # Slash commands
│   ├── dev.md
│   ├── bmad-pilot.md
│   └── ...
├── agents/                  # Agent definitions
│   ├── bmad-po.md
│   ├── code.md
│   └── ...
├── skills/                  # Skills
│   ├── codeagent/
│   └── codex/
├── config.json              # Configuration
├── installed_modules.json   # Installation status
└── docs/                    # Documentation
    ├── CODEAGENT-WRAPPER.md
    └── ...
```

## Appendix D: Troubleshooting

### Installation Issues

**Permission denied**:
```bash
python3 install.py --install-dir ~/.claude --force
```

**Module not found**:
```bash
# Check available modules
python3 install.py --list-modules

# Verify config.json syntax
python3 -c "import json; json.load(open('config.json'))"
```

### codeagent-wrapper Issues

**Backend not found**:
```bash
# Verify backend installation
which codex
which claude
which gemini

# Check PATH
echo $PATH
```

**Session resume failed**:
```bash
# Check session history
codex history
claude history

# Verify session ID format
codeagent-wrapper resume <session_id> "continue"
```

**Parallel tasks not running**:
```bash
# Verify task format
# Ensure ---TASK--- and ---CONTENT--- delimiters
# Check task IDs are unique
# Verify dependencies reference existing IDs
```

### Workflow Issues

**Quality score too low**:
- Provide more detailed descriptions
- Add specific requirements
- Include edge cases in scope

**Commands not found**:
```bash
# Verify installation
/plugin list

# Reinstall
make install
```

## Appendix E: Glossary

| Term | Definition |
|------|------------|
| **BMAD** | Business-Minded Agile Development - full 6-phase workflow |
| **Requirements Workflow** | Lightweight 4-phase workflow |
| **Dev Workflow** | Minimal concurrent development workflow |
| **codeagent-wrapper** | CLI tool for multi-backend execution |
| **Backend** | AI provider (Codex, Claude, Gemini) |
| **Skill** | Claude Code capability module |
| **Hook** | Event-triggered script |
| **Agent** | Specialized AI role |
| **Quality Gate** | Minimum score required to proceed |
| **Session** | Continued conversation context |

## Appendix F: Related Resources

- [Codex CLI Documentation](https://codex.docs)
- [Claude CLI Documentation](https://claude.ai/docs)
- [Gemini CLI Documentation](https://ai.google.dev/docs)
- [Claude Code Hooks](https://docs.anthropic.com/claude-code/hooks)
- [Claude Code Plugins](https://docs.anthropic.com/claude-code/plugins)

---

**Version**: 5.2.0
**Last Updated**: 2026-01-03
**License**: AGPL-3.0
