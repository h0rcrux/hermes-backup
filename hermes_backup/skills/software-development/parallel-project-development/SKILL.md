---
name: parallel-project-development
description: Develop multi-module projects using parallel subagents, with GitHub as the deployment target. Avoid local deployment in resource-constrained environments.
tags: [subagent, parallel, github, development-workflow]
---

# Parallel Project Development with GitHub Deployment

## Key Principle
**Do NOT deploy locally in Docker containers.** Generate code → Push to GitHub → User pulls to their machine for testing. This saves container resources.

## Workflow Pattern

### 1. Project Scaffolding (Main Agent)
- Create project structure and shared files
- Initialize Git repository early
- Create `.gitignore` before first commit

### 2. Parallel Module Development (Subagents)
Use `delegate_task` with parallel tasks for independent modules:

```python
delegate_task(
    tasks=[
        {"goal": "Create module A", "context": "...", "toolsets": ["file", "terminal"]},
        {"goal": "Create module B", "context": "...", "toolsets": ["file", "terminal"]},
        {"goal": "Create module C", "context": "...", "toolsets": ["file", "terminal"]},
    ]
)
```

**Dependency ordering:**
- Batch 1: Foundation modules (no dependencies)
- Batch 2: Modules depending on Batch 1
- Batch 3: Modules depending on Batch 2
- Continue as needed

### 3. Git Setup & GitHub Push

#### Configure Git (if not set)
```bash
git config --global user.name "Username"
git config --global user.email "email@example.com"
```

#### Initialize and Commit
```bash
cd /project-dir
git init
git add .
git commit -m "Initial commit: project description"
```

#### GitHub Authentication (when gh CLI unavailable)
Use token directly in remote URL:
```bash
# Set remote with token
git remote add origin https://username:ghp_TOKEN@github.com/username/repo.git

# Push
git push -u origin master

# IMPORTANT: Reset URL after push (remove token from config)
git remote set-url origin https://github.com/username/repo.git
```

#### GitHub Token Generation (instructions for user)
1. Visit https://github.com/settings/tokens
2. Generate new token (classic)
3. Select `repo` scope
4. Copy token (shown only once)

### 4. Clean Up After Push
```bash
# Stop all services
pkill -f "node"
pkill -f "npm"
pkill -f "streamlit"

# Remove node_modules to save space
rm -rf node_modules
rm -rf */node_modules
```

## Parallel Development Example

### Quant Trading System Example
```
Batch 1: trading-common (foundation)
Batch 2: market-data-service, indicator-service, strategy-core (parallel)
Batch 3: backtest-service, web-service (parallel)
Batch 4: web-app (frontend)
```

### Timing
- Each subagent: 5-30 minutes depending on complexity
- Max concurrent: 3 subagents (default)
- Total time saved: ~60-70% vs sequential

## Subagent Context Template
```python
{
    "goal": "Create [module-name] with [key features]",
    "context": """
## Project Dependencies
- dependency-a: already created (description)
- dependency-b: already created (description)

## Module Responsibilities
- What this module does
- Key interfaces it implements/exposes

## Technical Requirements
- Language/Framework
- Key dependencies
- Port number (if service)

## File Structure
path/to/module/
├── file1.ext
├── file2.ext
└── subdir/
    └── file3.ext

## Implementation Details
- Specific algorithms
- API endpoints
- Data formats
""",
    "toolsets": ["file", "terminal"]
}
```

## Post-Development Checklist
- [ ] All modules created
- [ ] Git repository initialized
- [ ] All files committed
- [ ] GitHub repository created (user)
- [ ] Code pushed to GitHub
- [ ] Remote URL cleaned (token removed)
- [ ] Local services stopped
- [ ] Temporary files cleaned

## What NOT To Do
- ❌ Don't run `npm install` in container
- ❌ Don't start development servers in container
- ❌ Don't run databases in container
- ❌ Don't keep node_modules after development
- ❌ Don't deploy services locally

## What TO Do
- ✅ Generate complete, runnable code
- ✅ Include setup instructions in README
- ✅ Create start/stop scripts for user
- ✅ Push to GitHub immediately
- ✅ Clean up resources after push
