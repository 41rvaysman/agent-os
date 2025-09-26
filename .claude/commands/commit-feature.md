# Commit Feature

Complete feature development and merge changes to main branch. This command commits your changes, merges to main, and cleans up the feature branch.

```bash
# Commit changes with message from commit message section
git add .
git commit -m "Add create feature command functionality"

# Switch to main branch
git checkout main

# Merge feature branch to main
git merge feature/add-create-feature-command

# Push changes to origin main
git push origin main

# Clean up feature branch
git branch -d feature/add-create-feature-command
git push origin --delete feature/add-create-feature-command
```

## Commit Message
Add create feature command functionality

This command performs the following steps:
1. Commits all staged changes with the specified commit message
2. Switches to the main branch
3. Merges the feature branch into main
4. Pushes changes to origin main
5. Deletes the local feature branch
6. Deletes the remote feature branch

