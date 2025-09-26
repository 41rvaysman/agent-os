# Add New Feature $ARGUMENTS

Set up a new feature branch for development. This command will ensure you start from the latest main branch and create a properly named feature branch.

```bash
# Start from your main branch
git checkout main
git pull origin main

# Create and switch to feature branch
git checkout -b feature/$ARGUMENTS

# Confirm the branch was created successfully
git branch --show-current
```

This command performs the following steps:
1. Switches to the main branch
2. Pulls the latest changes from origin/main
3. Creates and switches to a new feature branch named `feature/$ARGUMENTS`
4. Confirms the new branch is active
