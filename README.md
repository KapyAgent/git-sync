# git-sync

A lightweight utility to real-time synchronize changes from one Git worktree to another, designed for workflows where you want to test feature branch changes against a "main" or "staging" environment without constant manual commits or `rsync` calls.

## Features

- **Real-time Sync**: Uses `watchdog` (inotify/FSEvents) to monitor file changes.
- **Git-Aware**: Automatically respects `.gitignore` in both source and destination.
- **Fatal on Conflict**: If a file is unignored in the source but ignored in the destination, the script exits immediately with an error and cleans up the destination.
- **Safety First**: Verifies that both directories belong to the same Git repository before starting.
- **Auto-Cleanup**: Automatically performs `git reset --hard` and `git clean -fd` on the destination directory upon startup and exit.
- **Wrapper Mode**: Can execute a command (e.g., a build script or dev server) and keep syncing until the command finishes or is interrupted.
- **Robust Process Management**: When a fatal error occurs or the user interrupts, it forcefully kills the child process and its entire process group before cleaning up.
- **Shell Support**: Supports complex shell commands (pipes, redirections, heredocs) when wrapped in quotes.
- **Efficient Initialization**: Uses `git ls-files` for fast initial synchronization (v3+).

## Installation

Ensure you have [uv](https://github.com/astral-sh/uv) installed.

```bash
git clone https://github.com/KapyAgent/git-sync.git
cd git-sync
chmod +x git-sync
```

## Usage

```bash
./git-sync <src_dir> <dest_dir> [command...]
```

### Examples

**Basic synchronization (running from the destination directory):**
```bash
cd ./main-branch && ../git-sync ../feat-branch .
```

**Run a command while syncing:**
```bash
cd ./main-branch && ../git-sync ../feat-branch . "npm run dev"
```

**Complex shell commands:**
```bash
cd ./main-branch && ../git-sync ../feat-branch . "make | tee build.log"
```

## How It Works (Principles)

1. **Initialization**:
   - Validates that `<src>` and `<dest>` are directories and share the same Git common directory.
   - Performs an initial cleanup of the `<dest>` directory to ensure a clean state.
   - **Batch Sync**: Executes an initial recursive sync of all non-ignored files from `<src>` to `<dest>` using `git ls-files` and `shutil.copy2` for maximum performance.
   - Prints "Initial sync completed." upon success.

2. **Monitoring**:
   - Starts a background observer using the `watchdog` library.
   - Monitors for `created`, `modified`, `deleted`, and `moved` events in the `<src>` directory.

3. **Filtering & Fatal Errors**:
   - For every change, it checks `git check-ignore`. 
   - Files ignored in `<src>` are skipped.
   - **Fatal Logic**: If a file is unignored in `<src>` but ignored in `<dest>`, the script prints a `FATAL ERROR`, stops the observer, kills any running wrapper command (including its entire process group), performs final cleanup on `<dest>`, and exits with code `1`.

4. **Execution**:
   - If a command is provided, it runs it in a subshell (`shell=True`) within its own process group (`start_new_session=True`).
   - The script waits for the command to finish.

5. **Cleanup**:
   - On exit (due to fatal error, command completion, or signals like `Ctrl+C`), it stops the observer, terminates child processes, and restores the `<dest>` directory to its original Git state using `git reset --hard` and `git clean -fd`.

## Requirements

- Python 3.12+
- `uv` (for dependency management)
- `git`
