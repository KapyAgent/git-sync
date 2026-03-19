# git-sync

A Python script to synchronize Git worktree changes between branches in real-time.

## Features
- **Real-time Sync**: Uses `watchdog` to monitor the source directory and sync changes to the destination directory immediately.
- **Gitignore Respect**: Only synchronizes files that are not ignored by `.gitignore` in both source and destination.
- **Safety Checks**: Verifies that both source and destination directories belong to the same Git repository.
- **Wrapper Mode**: Can execute a command while syncing and clean up automatically upon command exit or interruption.
- **Efficient Initialization**: Uses `git ls-files` for fast initial synchronization.
- **Automatic Cleanup**: Resets and cleans the destination directory upon exit.

## Installation
Ensure you have `uv` installed, then:
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
To monitor changes and run a command in the destination directory:
```bash
./git-sync ./feat-branch ./main-branch "cd ./main-branch && make build"
```

## Principles
1. **Initialization**: On startup, the script performs a hard reset and clean on the destination directory. It then identifies all non-ignored files in the source and copies them to the destination.
2. **Monitoring**: Uses `watchdog` to listen for file system events (create, modify, delete, move) in the source directory.
3. **Filtering**: Each event is checked against `.gitignore` rules for both source and destination to ensure consistency.
4. **Execution**: If a command is provided, it is executed in a new process group. The script waits for the command to finish.
5. **Fatal Errors**: If a sync conflict (e.g., a file being synced is ignored in the destination) is detected during monitoring, the script triggers a fatal error, kills the command process group, cleans up the destination, and exits with code 1.
6. **Cleanup**: When the script exits (either normally or via Ctrl+C), it ensures the destination directory is reset to its original clean state.

## Requirements
- Python >= 3.12
- `uv` (for dependency management and execution)
- Git
