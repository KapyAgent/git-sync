# git-sync

A lightweight utility to real-time synchronize changes from one Git worktree to another, designed for workflows where you want to test feature branch changes against a "main" or "staging" environment without constant manual commits or `rsync` calls.

## Features

- **Real-time Sync**: Uses `watchdog` (inotify/FSEvents) to monitor file changes.
- **Git-Aware**: Automatically respects `.gitignore` in both source and destination.
- **Safety First**: Verifies that both directories belong to the same Git repository before starting.
- **Auto-Cleanup**: Automatically performs `git reset --hard` and `git clean -fd` on the destination directory upon startup and exit.
- **Wrapper Mode**: Can execute a command (e.g., a build script or dev server) and keep syncing until the command finishes or is interrupted.
- **Shell Support**: Supports complex shell commands (pipes, redirections, heredocs) when wrapped in quotes.

## Installation

Ensure you have [uv](https://github.com/astral-sh/uv) installed.

```bash
git clone https://github.com/KapyAgent/git-sync.git
cd git-sync
chmod +x git-sync
# Optional: Add to your PATH
# export PATH="$PATH:$(pwd)"
```

## Usage

```bash
./git-sync <src_dir> <dest_dir> [command...]
```

### Examples

**Basic synchronization:**
```bash
./git-sync ./feat-branch ./main-branch
```

**Run a command while syncing:**
```bash
./git-sync ./feat-branch ./main-branch "npm run dev"
```

**Complex shell commands:**
```bash
./git-sync ./feat-branch ./main-branch "make | tee build.log"
```

## How It Works (Principles)

1. **Initialization**:
   - Validates that `<src>` and `<dest>` are directories and share the same Git common directory.
   - Performs an initial cleanup of the `<dest>` directory to ensure a clean state.
   - Executes an initial recursive sync of all non-ignored files from `<src>` to `<dest>`.

2. **Monitoring**:
   - Starts a background observer using the `watchdog` library.
   - Monitors for `created`, `modified`, `deleted`, and `moved` events in the `<src>` directory.

3. **Filtering**:
   - For every change, it checks `git check-ignore`. 
   - Files ignored in `<src>` are skipped.
   - If a file is unignored in `<src>` but ignored in `<dest>`, it prints a fatal error (but continues monitoring other files) to prevent accidental overwrites of destination-specific ignored files.

4. **Execution**:
   - If a command is provided, it runs it in a subshell (`shell=True`).
   - The script waits for the command to finish.

5. **Cleanup**:
   - On exit (due to command completion or signals like `Ctrl+C`), it stops the observer and restores the `<dest>` directory to its original Git state using `git reset --hard` and `git clean -fd`.

## Requirements

- Python 3.12+
- `uv` (for dependency management)
- `git`
