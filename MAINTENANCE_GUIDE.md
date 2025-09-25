# AgentOS Fork Maintenance Guide

## Feature Branch Workflow for Custom Changes

### Creating New Features/Modifications
```bash
# Start from your main branch
git checkout main
git pull origin main

# Create and switch to feature branch
git checkout -b feature/your-feature-name

# Make your changes
# ... edit files, add agents, modify standards ...

# Commit your changes
git add .
git commit -m "Descriptive commit message"

# Switch back to main and merge
git checkout main
git merge feature/your-feature-name

# Push to your fork
git push origin main

# Clean up feature branch (optional)
git branch -d feature/your-feature-name
```

### Branch Naming Conventions
- `feature/add-new-agent` - Adding new custom agents
- `feature/update-standards` - Modifying your custom standards
- `feature/custom-instructions` - Updating instruction files
- `fix/agent-bug` - Bug fixes in your custom code
- `update/project-config` - Configuration updates

### Benefits of Feature Branches
- ✅ Keeps main branch stable and clean
- ✅ Easier upstream merges (less conflicts)
- ✅ Safe experimentation and easy rollbacks
- ✅ Clear history of related changes
- ✅ Better collaboration if working with others

## Regular Update Workflow (Every 1-2 Months)

### Step 1: Prepare for Update
```bash
# Ensure you're on main branch and committed all changes
git status
git checkout main
```

### Step 2: Sync Upstream Changes
```bash
# Switch to upstream-sync branch
git checkout upstream-sync

# Pull latest changes from original repository
git pull upstream main

# Push updated upstream-sync to your fork (optional, for backup)
git push origin upstream-sync
```

### Step 3: Merge to Your Main Branch
```bash
# Switch back to your main branch
git checkout main

# Merge upstream changes with no fast-forward (preserves history)
git merge upstream-sync --no-ff
```

### Step 4: Resolve Conflicts (if any)
Follow the directory-specific strategies below.

### Step 5: Finalize Update
```bash
# Push updated main branch to your fork
git push origin main
```

## Conflict Resolution Strategies

### General Approach
When conflicts occur, you'll see files marked with conflict markers:
```
<<<<<<< HEAD
Your changes
=======
Upstream changes
>>>>>>> upstream-sync
```

### Directory-Specific Handling

#### `agents/` Directory
**Strategy:** Accept most upstream agents, keep your custom ones

```bash
# Accept upstream for agents you want to keep updated
git checkout upstream-sync -- agents/new-agent.md
git checkout upstream-sync -- agents/existing-agent.md

# Keep your custom agents
git checkout main -- agents/pr-specialist.md
git checkout main -- agents/commit-specialist.md

# Remove replaced agents if they reappear
git rm agents/git-workflow.md

# Stage all changes
git add agents/
```

#### `standards/` Directory
**Strategy:** Keep your custom standards, evaluate new upstream categories

```bash
# Keep all your custom code-style files
git checkout main -- standards/code-style/

# Accept new upstream standard categories (evaluate first)
git checkout upstream-sync -- standards/new-category/

# Stage changes
git add standards/
```

#### `instructions/` Directory
**Strategy:** Manual merge to preserve your agent references

For each conflicted file:
1. Open the file in your editor
2. Keep your agent references (pr-specialist, commit-specialist)
3. Integrate new upstream features/improvements
4. Remove conflict markers
5. Save and stage the file

#### Root Files (`config.yml`, `README.md`, etc.)
**Strategy:** Usually accept upstream, then re-apply your configurations

```bash
# Accept upstream version
git checkout upstream-sync -- config.yml

# Edit file to re-add your custom configurations
# Then stage the file
git add config.yml
```

## Common Update Scenarios

### Scenario 1: New Agent Added Upstream
```bash
# New agent appears - usually accept it
git checkout upstream-sync -- agents/new-helpful-agent.md
git add agents/new-helpful-agent.md
```

### Scenario 2: Existing Agent Updated Upstream
```bash
# Agent you use got improvements - accept the update
git checkout upstream-sync -- agents/context-fetcher.md
git add agents/context-fetcher.md
```

### Scenario 3: Instruction File Modified
```bash
# Manual merge required - open in editor
# Keep your agent references, add new upstream features
# Remove conflict markers and save
git add instructions/core/modified-file.md
```

### Scenario 4: Your Custom Agent Conflicts
```bash
# Keep your version
git checkout main -- agents/pr-specialist.md
git add agents/pr-specialist.md
```

## Quick Command Reference

### Basic Update Commands
```bash
git checkout upstream-sync && git pull upstream main
git checkout main && git merge upstream-sync --no-ff
git push origin main
```

### Conflict Resolution Commands
```bash
# Accept upstream version for specific file
git checkout upstream-sync -- path/to/file

# Keep your version for specific file
git checkout main -- path/to/file

# Remove file that shouldn't exist
git rm path/to/file

# Stage all changes in directory
git add directory-name/
```

### Status and Information Commands
```bash
# Check current status
git status

# See conflicted files
git diff --name-only --diff-filter=U

# View branches and remotes
git branch -a
git remote -v
```

## Troubleshooting

### Problem: Merge Conflicts Seem Overwhelming
**Solution:** Handle one directory at a time:
1. Resolve all conflicts in `agents/` first
2. Then `standards/`
3. Then `instructions/`
4. Finally root files

### Problem: Lost Track of Which Files to Keep
**Solution:** Reference your custom modifications:
- Your agents: `pr-specialist.md`, `commit-specialist.md`
- Your standards: Everything in `standards/code-style/`
- Modified instructions: Files referencing your custom agents

### Problem: Accidentally Accepted Wrong Version
**Solution:** Before committing, you can still fix it:
```bash
# Reset specific file to your version
git checkout main -- path/to/file
git add path/to/file
```

### Problem: Want to See What Changed Upstream
**Solution:** Compare upstream changes:
```bash
# See what's new in upstream
git log main..upstream-sync --oneline

# See file differences
git diff main upstream-sync -- path/to/file
```

## Best Practices

### For Feature Branches
1. **Always start from updated main** - `git checkout main && git pull origin main`
2. **Use descriptive branch names** - Makes it clear what you're working on
3. **Keep feature branches focused** - One feature/fix per branch
4. **Test before merging** - Ensure your changes work as expected
5. **Clean up merged branches** - Delete feature branches after merging

### For Upstream Updates
1. **Always commit your work** before starting an update
2. **Review upstream changes** before merging (check their changelog)
3. **Test your setup** after updates to ensure everything works
4. **Keep notes** on which upstream agents you specifically want to avoid
5. **Backup important customizations** outside the repo if they're critical

### Combined Workflow
- Use feature branches for your custom work
- Keep main branch clean for upstream merges
- Never work directly on upstream-sync branch

## Emergency Reset

If something goes wrong during merge:
```bash
# Abort the merge and start over
git merge --abort

# Reset to last known good state
git reset --hard origin/main
```

Then start the update process again more carefully.

## Maintenance Schedule

**Recommended frequency:** Every 1-2 months or when major upstream features are announced

**Pre-update checklist:**
- [ ] All local changes committed
- [ ] Review upstream changelog/releases
- [ ] Backup any critical custom configurations
- [ ] Set aside time for potential conflict resolution