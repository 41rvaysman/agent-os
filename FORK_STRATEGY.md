# Fork Management Strategy

## Overview
This document outlines the strategy for maintaining a custom fork of the AgentOS repository while staying synchronized with upstream updates.

## Repository Setup

### Initial Setup
1. **Fork the repo** to your GitHub account
2. **Create branches:**
   ```bash
   git checkout -b upstream-sync  # tracks original repo
   git checkout -b main          # your custom version (default)
   ```

3. **Add upstream remote:**
   ```bash
   git remote add upstream https://github.com/buildermethods/agent-os.git
   ```

## Update Workflow (Every 1-2 Months)

### 1. Sync Upstream Changes
```bash
git checkout upstream-sync
git pull upstream main
```

### 2. Selective Merge to Main Branch
```bash
git checkout main
git merge upstream-sync --no-ff
```

### 3. Handle Conflicts Strategically

#### Auto-Accept Upstream
- Core framework files you didn't modify
- Root configuration files (re-apply your configs after)
- Documentation updates

#### Keep Your Version
- Custom agents: `pr-specialist.md`, `commit-specialist.md`
- Custom standards in `standards/` directory
- Any completely replaced files

#### Manual Merge
- Modified instruction files (to integrate new features while keeping your agent references)
- Files where you want both upstream improvements and your customizations

## Directory-Specific Strategies

### `agents/` Directory
- **Take upstream:** New agents and updates to existing agents (95% usage)
- **Keep yours:** Custom `pr-specialist.md` and `commit-specialist.md`
- **Skip/Remove:** Original `git-workflow.md` (replaced by your custom agents)

**Conflict resolution commands:**
```bash
# Accept upstream for agents you want to keep updated
git checkout upstream-sync -- agents/new-agent.md
git checkout upstream-sync -- agents/existing-agent.md

# Keep your custom agents
git checkout main -- agents/pr-specialist.md
git checkout main -- agents/commit-specialist.md

# Remove replaced agents if they reappear
git rm agents/git-workflow.md
```

### `standards/` Directory
- **Keep yours:** All files in `standards/code-style/` (completely different content)
- **Evaluate upstream:** New standard categories that might be useful

### `instructions/` Directory
- **Manual merge:** Modified files to integrate new features while preserving your agent references
- **Take upstream:** New instruction files that don't conflict with your modifications

### Root Files
- **Usually take upstream:** `config.yml`, `README.md`, core framework files
- **Re-apply your configurations** after accepting upstream changes

## Conflict Prevention

### Optional: Prevent Resurrection of Replaced Files
```bash
echo "claude-code/agents/git-workflow.md" >> .gitignore
```

## Benefits of This Strategy

- ✅ Get all upstream improvements and new features
- ✅ Maintain your customizations intact
- ✅ Clear separation between original and custom content
- ✅ Batch updates on your preferred schedule (1-2 months)
- ✅ Selective adoption of upstream changes
- ✅ Preserve your custom agents while getting new upstream agents

## Workflow Summary

1. **Sync upstream** changes to `upstream-sync` branch
2. **Merge selectively** to your `main` branch
3. **Resolve conflicts** using directory-specific strategies
4. **Test** your custom setup with new upstream changes
5. **Commit** the integrated updates

This approach balances staying current with upstream while preserving your custom modifications and project-specific needs.