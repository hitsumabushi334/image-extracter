# Project Structure

## Root Directory Organization

```
/workspace/
├── .claude/                    # Claude Code configuration
│   ├── agents/                 # Specialized AI agents
│   ├── commands/               # Slash command definitions
│   │   └── kiro/              # Kiro-specific commands
│   ├── settings.json          # Core configuration
│   └── settings.local.json    # Local overrides
├── .kiro/                     # Kiro framework files
│   ├── steering/              # Project guidance documents
│   └── specs/                 # Feature specifications
├── CLAUDE.md                  # Project instructions
└── fastmcp_deepresearch.md    # Research documentation
```

## Subdirectory Structures

### `.claude/` - Claude Code Configuration
Core configuration for the AI development environment:

- **`agents/`**: Specialized AI agents for different concerns
  - `research-specialist.md`: Research and investigation
  - `architecture-design-reviewer.md`: Design validation  
  - `code-quality-reviewer.md`: Quality assurance
  - `security-code-reviewer.md`: Security analysis
  - `refactoring-code-reviewer.md`: Refactoring guidance
  - `kiro-task-implementer.md`: Task execution

- **`commands/`**: Slash command definitions
  - `check-task-list.md`: Task tracking
  - `codeReview.md`: Code review automation
  - `create-commit.md`: Git operations
  - `deepResearch.md`: Research workflows
  - `git-create-branch.md`: Branch management
  - `git-create-pr.md`: PR creation
  - `serena.md`: Token-efficient operations

- **`commands/kiro/`**: Kiro-specific command set
  - `steering.md`: Project guidance management
  - `steering-custom.md`: Custom steering
  - `spec-*.md`: Specification workflow commands

### `.kiro/` - Framework Core
Kiro-specific project files:

- **`steering/`**: Project-wide guidance documents
  - `product.md`: Business context and objectives
  - `tech.md`: Technology stack and architecture
  - `structure.md`: File organization patterns
  - Custom steering files (as needed)

- **`specs/`**: Feature specification documents
  - Individual feature specification folders
  - Requirements, design, and task documents
  - Progress tracking files

## Code Organization Patterns

### Command Structure
All commands follow the frontmatter pattern:
```markdown
---
description: Command description
allowed-tools: Tool1, Tool2, Tool3
---
# Command Implementation
```

### Agent Structure  
Agents use standardized metadata:
```markdown
---
name: agent-name
description: Agent purpose and usage
tools: Available tool list
color: UI color indicator
---
# Agent Implementation
```

### Configuration Hierarchy
1. **Global settings**: `settings.json`
2. **Local overrides**: `settings.local.json`  
3. **Project instructions**: `CLAUDE.md`
4. **Steering guidance**: `.kiro/steering/*.md`

## File Naming Conventions

### Command Files
- **Kebab-case**: `git-create-pr.md`, `check-task-list.md`
- **Namespace separation**: `kiro/spec-requirements.md`
- **Descriptive names**: Clear purpose indication

### Agent Files
- **Kebab-case**: `research-specialist.md`
- **Purpose-based**: `code-quality-reviewer.md`
- **Role indication**: `kiro-task-implementer.md`

### Configuration Files
- **Standard names**: `settings.json`, `CLAUDE.md`
- **Local variants**: `settings.local.json`
- **Documentation**: `README.md`, `*.md` for docs

### Steering Files
- **Functional names**: `product.md`, `tech.md`, `structure.md`
- **Custom naming**: Descriptive purpose-based names
- **Lowercase**: Consistent with markdown conventions

## Import Organization

### Command Dependencies
Commands specify their required tools in frontmatter:
```markdown
allowed-tools: Bash, Read, Write, Edit, MultiEdit
```

### Agent Tools
Agents declare available tools:
```markdown
tools: Task, WebSearch, Context7, HumanInTheLoop
```

### Hook Integration
Settings define hook execution patterns:
```json
{
  "hooks": {
    "PreToolUse": [...],
    "PostToolUse": [...],
    "Stop": [...],
    "Notification": [...]
  }
}
```

## Key Architectural Principles

### Separation of Concerns
- **Commands**: Specific development actions
- **Agents**: Specialized AI capabilities  
- **Steering**: Project-wide guidance
- **Specs**: Feature-specific documentation

### Configuration Layering
- **Base configuration**: Core Claude Code settings
- **Local customization**: Environment-specific overrides
- **Project context**: CLAUDE.md instructions
- **Dynamic guidance**: Steering documents

### Phase-Based Workflow
1. **Steering**: Project guidance and context
2. **Requirements**: Feature specification
3. **Design**: Architecture and approach
4. **Tasks**: Implementation breakdown
5. **Implementation**: Actual development

### Human-in-the-Loop
- **Approval gates**: Human confirmation at each phase
- **Interactive prompts**: User input for key decisions
- **Notification system**: Discord integration for updates
- **Review processes**: Mandatory human oversight

### Documentation-Driven Development
- **Specification first**: Document before implementing
- **Version controlled**: All guidance in git
- **Living documents**: Updated with project evolution
- **Searchable knowledge**: Structured information storage