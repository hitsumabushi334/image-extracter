# Technology Stack

## Architecture

Kiro operates as a meta-development framework built entirely on Claude Code's command and agent system. The architecture follows a modular, agent-based design where specialized AI components handle different aspects of the development lifecycle.

### System Architecture
- **Core Framework**: Claude Code CLI with custom slash commands
- **Agent System**: Specialized AI agents for different development concerns
- **Hook System**: Pre/post tool execution hooks for automation
- **Configuration Management**: JSON-based settings with local overrides

## Development Environment

### Required Tools
- **Claude Code CLI**: Primary development interface
- **Git**: Version control (hooks configured for commit logging)
- **Bash**: Shell automation and scripting
- **MCP Servers**: Model Context Protocol servers for external integrations

### Optional Integrations
- **FastMCP**: Python library for MCP server development
- **Context7**: Library documentation and code examples
- **Discord**: Notification system (configured in hooks)
- **Black**: Python code formatting (automated via hooks)

## Command System

### Core Commands (`/.claude/commands/`)
- **check-task-list.md**: Task progress tracking
- **codeReview.md**: Code review automation
- **create-commit.md**: Git commit creation
- **deepResearch.md**: Comprehensive research workflows
- **git-create-branch.md**: Branch management
- **git-create-pr.md**: Pull request creation
- **serena.md**: Token-efficient problem solving

### Kiro-Specific Commands (`/.claude/commands/kiro/`)
- **steering.md**: Project guidance management
- **steering-custom.md**: Custom steering configuration
- **spec-init.md**: Specification initialization
- **spec-requirements.md**: Requirements documentation
- **spec-design.md**: Design specification
- **spec-tasks.md**: Task breakdown
- **spec-status.md**: Progress tracking

## Agent System

### Specialized Agents (`/.claude/agents/`)
- **research-specialist.md**: Comprehensive research and investigation
- **architecture-design-reviewer.md**: System design validation
- **code-quality-reviewer.md**: Code quality assurance
- **security-code-reviewer.md**: Security analysis
- **refactoring-code-reviewer.md**: Code refactoring guidance
- **kiro-task-implementer.md**: Task execution automation

## Configuration System

### Settings Files
- **settings.json**: Core Claude Code configuration with security permissions
- **settings.local.json**: Local environment overrides
- **CLAUDE.md**: Project-specific instructions and rules

### Hook Configuration
- **PreToolUse**: Command validation and security checks
- **PostToolUse**: Automated formatting and logging
- **Stop/Notification**: Discord integration for workflow notifications

## Development Commands

### Common Workflows
```bash
# Initialize new feature specification
/kiro:spec-init [feature-description]

# Update project guidance
/kiro:steering

# Conduct research
/deepResearch [topic]

# Code review process
/codeReview

# Task management
/check-task-list
```

### Security Permissions
- Bash command restrictions via deny list
- Pre-execution validation hooks
- Command history logging
- File modification tracking

## Environment Variables

### Core Configuration
- **CLAUDE_API_KEY**: Claude API authentication
- **USER**: System user identification
- **HOME**: User directory for scripts and logs

### Hook Integration
- **Discord webhook URLs**: Notification system
- **Script paths**: Custom automation scripts
- **Log file locations**: Command history and audit trails

## Port Configuration

No traditional server ports required - operates entirely through Claude Code CLI interface and file system operations.

## External Dependencies

### MCP Servers
- **Human-in-the-loop**: Interactive user confirmation
- **Context7**: Library documentation access
- **FastMCP**: Python development support
- **Playwright**: Browser automation (if needed)
- **Serena**: Semantic code analysis

### Integration APIs
- **Web Search**: Real-time information gathering
- **Git**: Version control integration
- **Discord**: Team notification system