---
name: github-multi-version-repo
description: Organize multiple project versions (v1/v2/v3) in a single GitHub repository with unified documentation
tags: [github, versioning, multi-version, project-organization]
---

# GitHub Multi-Version Repository

Organize multiple project versions in a single GitHub repository using directory-based versioning.

## Pattern

```
repo-root/
├── README.md                    # Unified docs with version comparison
├── VERSION_COMPARISON.md        # Detailed feature comparison
├── v1/                          # Version 1
├── v2/                          # Version 2
└── v3/                          # Version 3
```

## Implementation Steps

### 1. Create unified directory
```bash
mkdir /opt/data/project-all
```

### 2. Copy versions with exclusions
```python
import shutil

def ignore_patterns(dir, files):
    return ['node_modules', '.git', '__pycache__', 'dist', 'logs', '*.zip']

shutil.copytree(v1_src, "v1", ignore=ignore_patterns)
shutil.copytree(v2_src, "v2", ignore=ignore_patterns)
```

### 3. Create unified README
Include:
- Version comparison table
- Quick start for each version
- Feature matrix
- Version selection guide

### 4. Push to existing repo (overwrite)
```bash
git init
git add .
git commit -m "feat: multi-version repo"
git remote add origin https://TOKEN@github.com/user/repo.git
git push -u origin master --force
```

## GitHub Token Auth (when gh CLI unavailable)

```python
github_token = "ghp_..."
remote_url = f"https://{username}:{github_token}@github.com/{username}/{repo}.git"
subprocess.run(["git", "remote", "set-url", "origin", remote_url])
# After push, restore URL without token
subprocess.run(["git", "remote", "set-url", "origin", original_url])
```

## Key Learnings

1. **Force push** to overwrite existing repo when consolidating versions
2. **Exclude** node_modules, .git, __pycache__ to reduce size
3. **Unified README** with version comparison helps users choose
4. **VERSION_COMPARISON.md** for detailed feature matrix
5. **Restore remote URL** after push to remove token from config

## Pitfalls

- Force push destroys history - warn user first
- Large repos may timeout - exclude unnecessary files
- Token in remote URL is security risk - always restore after push
