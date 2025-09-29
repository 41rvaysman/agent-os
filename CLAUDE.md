# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

AgentOS is a spec-driven agentic development system that transforms AI coding agents into productive developers through structured workflows, specifications, and standards. It provides a comprehensive framework for managing product development using AI agents.

## Key Architecture Components

### Core Workflow System
The repository implements a multi-stage development workflow orchestrated through instruction files:

1. **Pre-flight checks** (`instructions/meta/pre-flight.md`) - Initial validations before task execution
2. **Product analysis** (`instructions/core/analyze-product.md`) - Understanding the product context
3. **Spec creation** (`instructions/core/create-spec.md`) - Detailed specification generation
4. **Task planning** (`instructions/core/create-tasks.md`) - Breaking specs into executable tasks
5. **Task execution** (`instructions/core/execute-tasks.md`) - Implementation of planned tasks
6. **Post-execution** (`instructions/core/post-execution-tasks.md`) - Cleanup and validation
7. **Post-flight checks** (`instructions/meta/post-flight.md`) - Final validations

### Agent System
Located in `claude-code/agents/`, specialized agents handle different aspects of development:
- **context-fetcher** - Retrieves relevant project context
- **file-creator** - Creates and manages files
- **date-checker** - Determines current dates for naming
- **pr-specialist** - Manages pull request creation
- **commit-specialist** - Handles git commits
- **test-runner** - Executes and validates tests
- **code-documenter** - Generates documentation
- **project-manager** - Orchestrates development tasks
- **git-workflow** - Manages git operations

### Specification Structure
Specs are created in `.agent-os/specs/YYYY-MM-DD-spec-name/` with:
- `spec.md` - Full requirements document
- `spec-lite.md` - Condensed version for AI context
- `sub-specs/` - Technical, database, and API specifications as needed

## Common Development Commands

### Feature Branch Workflow
```bash
# Create new feature branch
git checkout main && git pull origin main
git checkout -b feature/feature-name

# Commit and merge to main
git add . && git commit -m "message"
git checkout main && git merge feature/feature-name
git push origin main
git branch -d feature/feature-name
```

### Custom Claude Commands
The `.claude/commands/` directory contains custom commands:
- `/add-feature [name]` - Creates a new feature branch
- `/commit-feature` - Commits, merges to main, and cleans up feature branch

### AgentOS Commands
Located in `commands/` directory:
- `/analyze-product` - Analyze product context
- `/create-spec` - Generate feature specification
- `/create-tasks` - Create task checklist from spec
- `/execute-tasks` - Execute planned tasks
- `/plan-product` - Plan product development

## Fork Maintenance Strategy

This repository follows a specific fork maintenance strategy (see `MAINTENANCE_GUIDE.md`):

1. **Feature branches** for custom changes - Never work directly on main
2. **upstream-sync branch** for tracking original repository updates
3. **Regular syncing** (monthly) from upstream to incorporate improvements
4. **Conflict resolution** priorities:
   - Keep custom agents in `claude-code/agents/`
   - Keep custom standards in `standards/`
   - Manually merge instruction file changes

## Important File Locations

- **Product context**: `.agent-os/product/` (mission-lite.md, tech-stack.md, roadmap.md)
- **Specifications**: `.agent-os/specs/YYYY-MM-DD-spec-name/`
- **Standards**: `standards/` (code-style.md, best-practices.md, tech-stack.md)
- **Custom agents**: `claude-code/agents/`
- **Instructions**: `instructions/core/` and `instructions/meta/`

## Development Notes

- This is a Git repository-based system requiring git for version control
- No traditional build/test commands - the system uses spec-driven development
- Agents use subagent references in instructions (e.g., `subagent="context-fetcher"`)
- Date formatting follows YYYY-MM-DD pattern for spec folders
- Maximum 5 words in kebab-case for spec naming conventions