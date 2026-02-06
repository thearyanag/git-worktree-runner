# gtr Configuration Reference

## Table of Contents
- Worktree Directory Settings
- Editor Settings
- AI Tool Settings
- File Copying (Patterns & Directories)
- .worktreeinclude File
- Hooks
- Configuration Examples
- Advanced Usage: Same-Branch Worktrees
- Shell Completions

## Worktree Directory Settings

```bash
# Base directory (default: <repo-name>-worktrees as sibling to repo)
gtr.worktrees.dir = <path>

# Supports: absolute paths, repo-relative paths, tilde expansion
gtr.worktrees.dir = /Users/you/all-worktrees/my-project   # absolute
gtr.worktrees.dir = .worktrees                              # repo-relative (add to .gitignore)
gtr.worktrees.dir = ~/worktrees/my-project                  # tilde expansion

# Folder prefix (default: "")
gtr.worktrees.prefix = dev-

# Default branch (default: auto-detect)
gtr.defaultBranch = main
```

If storing worktrees inside the repository, add the directory to `.gitignore`:
```bash
echo "/.worktrees/" >> .gitignore
```

## Editor Settings

```bash
gtr.editor.default = cursor  # options: cursor, vscode, zed, none
```

Setup:
- **Cursor**: Install from cursor.com, enable shell command
- **VS Code**: Install from code.visualstudio.com, enable `code` command
- **Zed**: Install from zed.dev, `zed` available automatically

## AI Tool Settings

```bash
gtr.ai.default = none  # options: aider, claude, codex, continue, cursor, gemini, opencode, none
```

| Tool | Install | Set Default |
|---|---|---|
| Aider | `pip install aider-chat` | `git gtr config set gtr.ai.default aider` |
| Claude Code | Install from claude.com | `git gtr config set gtr.ai.default claude` |
| Codex CLI | `npm install -g @openai/codex` | `git gtr config set gtr.ai.default codex` |
| Continue | See docs.continue.dev | `git gtr config set gtr.ai.default continue` |
| Cursor | Install from cursor.com | `git gtr config set gtr.ai.default cursor` |
| Gemini | `npm install -g @google/gemini-cli` | `git gtr config set gtr.ai.default gemini` |
| OpenCode | Install from opencode.ai | `git gtr config set gtr.ai.default opencode` |

Pass arguments to AI tools:
```bash
git gtr ai my-feature -- --model gpt-4
git gtr ai my-feature -- --plan "refactor auth"
```

## File Copying

### Glob Patterns (multi-valued via `add`)

```bash
git gtr config add gtr.copy.include "**/.env.example"
git gtr config add gtr.copy.include "**/CLAUDE.md"
git gtr config add gtr.copy.include "*.config.js"

git gtr config add gtr.copy.exclude "**/.env"
git gtr config add gtr.copy.exclude "**/secrets.*"
```

### Directory Copying

Copy entire directories to avoid reinstalling dependencies:

```bash
git gtr config add gtr.copy.includeDirs "node_modules"
git gtr config add gtr.copy.includeDirs ".venv"
git gtr config add gtr.copy.includeDirs "vendor"

# Exclude sensitive subdirs
git gtr config add gtr.copy.excludeDirs "node_modules/.cache"
git gtr config add gtr.copy.excludeDirs "node_modules/.npm"
git gtr config add gtr.copy.excludeDirs "node_modules/.*"
```

Use cases: JS (`node_modules`), Python (`.venv`), PHP (`vendor`), Go (build caches).

## .worktreeinclude File

Create `.worktreeinclude` in repo root as alternative to config:

```
# .worktreeinclude - .gitignore-style syntax
**/.env.example
**/CLAUDE.md
*.config.js
```

Patterns from `.worktreeinclude` merge with `gtr.copy.include` config.

## Hooks

```bash
# Post-create hooks (multi-valued, run in order)
git gtr config add gtr.hook.postCreate "npm install"
git gtr config add gtr.hook.postCreate "npm run build"

# Post-remove hooks
git gtr config add gtr.hook.postRemove "echo 'Cleaned up!'"
```

Environment variables in hooks: `REPO_ROOT`, `WORKTREE_PATH`, `BRANCH`.

Language-specific examples:
```bash
# Node (npm/pnpm): git gtr config add gtr.hook.postCreate "npm install"
# Python:          git gtr config add gtr.hook.postCreate "pip install -r requirements.txt"
# Ruby:            git gtr config add gtr.hook.postCreate "bundle install"
# Rust:            git gtr config add gtr.hook.postCreate "cargo build"
```

## Configuration Examples

### Minimal
```bash
git gtr config set gtr.worktrees.prefix "wt-"
git gtr config set gtr.defaultBranch "main"
```

### Full Node.js Project
```bash
git gtr config set gtr.worktrees.prefix "wt-"
git gtr config set gtr.editor.default cursor
git gtr config add gtr.copy.include "**/.env.example"
git gtr config add gtr.copy.include "**/.env.development"
git gtr config add gtr.copy.exclude "**/.env.local"
git gtr config add gtr.hook.postCreate "pnpm install"
git gtr config add gtr.hook.postCreate "pnpm run build"
```

### Global Defaults
```bash
git gtr config set gtr.editor.default cursor --global
git gtr config set gtr.ai.default claude --global
```

### Project Setup Script (.gtr-setup.sh)
```bash
#!/bin/sh
git gtr config set gtr.worktrees.prefix "dev-"
git gtr config set gtr.editor.default cursor
git gtr config add gtr.copy.include ".env.example"
git gtr config add gtr.copy.include "docker-compose.yml"
git gtr config add gtr.hook.postCreate "docker-compose up -d db"
git gtr config add gtr.hook.postCreate "npm install"
git gtr config add gtr.hook.postCreate "npm run db:migrate"
```

## Advanced: Same-Branch Worktrees

Use `--force` with `--name` to create multiple worktrees on the same branch:

```bash
git gtr new feature-auth
git gtr new feature-auth --force --name backend    # feature-auth-backend/
git gtr new feature-auth --force --name frontend   # feature-auth-frontend/
git gtr new feature-auth --force --name tests      # feature-auth-tests/
```

Risks: concurrent edits can cause conflicts. Best practice: only edit files in one worktree at a time; commit/stash before switching.

## Shell Completions

**Bash**: `source /path/to/completions/gtr.bash` (requires bash-completion v2)
**Zsh**: Copy `completions/_git-gtr` to fpath, source after `compinit`
**Fish**: `ln -s /path/to/completions/gtr.fish ~/.config/fish/completions/`
