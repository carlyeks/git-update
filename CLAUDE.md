# CLAUDE.md - AI Assistant Guide for git-update

## Project Overview

**git-update** is a Python utility that automatically updates git repositories at specified intervals. It's designed to be run periodically (e.g., at login or via cron) to keep multiple git repositories synchronized with their remotes.

**Repository:** git-update
**Primary Language:** Python 3
**License:** Apache License 2.0
**Author:** Carl Yeksigian (Copyright 2013)

## Codebase Structure

This is a minimal, focused project with the following structure:

```
.
├── git-update          # Main executable Python script
├── requirements.txt    # Python dependencies
├── README.md           # User-facing documentation
├── LICENSE             # Apache 2.0 license
├── CLAUDE.md          # AI assistant documentation
└── .gitignore         # Standard Python .gitignore
```

### Key Files

#### `git-update` (Primary Executable)
- **Location:** `/git-update` (root of repository)
- **Type:** Python 3 script with shebang (`#!/usr/bin/env python3`)
- **Purpose:** Main and only script that handles all repository update logic
- **Dependencies:**
  - `git` - GitPython library for git operations
  - `os` - File system operations
  - `re` - Regular expression parsing for frequency strings
  - `datetime` - Time calculations and comparisons

## Technical Architecture

### Core Functionality

The script follows a simple workflow:

1. **Read Configuration** (`repostorefresh()`)
   - Reads `~/.update.repos` to get list of repositories and update frequencies
   - Reads `~/.update.lastruns` to get last update timestamps
   - Determines which repos need updating based on time elapsed

2. **Update Repositories** (`update()`)
   - Checks if repository is bare (skips if true)
   - Checks if repository is dirty (skips and warns if true)
   - Performs `git pull` on the remote

3. **Record Last Run** (main execution)
   - Writes timestamps to `~/.update.lastruns` for successfully updated repos

### Configuration Files

The script relies on two user configuration files in the home directory:

#### `~/.update.repos`
Format: `<repo_path><TAB><frequency>`
```
~/git-update	1w
~/another-repo	2d
```

Frequency format supports:
- `y` - years (approximated as 365 days)
- `m` - months (approximated as 30 days)
- `w` - weeks
- `d` - days
- `h` - hours

Example: `1w2d` = 1 week and 2 days

#### `~/.update.lastruns`
Format: `<repo_path><TAB><timestamp>`
```
~/git-update	1703436000.123456
```

Timestamps are in Unix epoch format with microseconds.

### Key Functions

#### `parsefrequency(frequency: str) -> timedelta`
- **Location:** git-update:31-49
- **Purpose:** Parses frequency strings (e.g., "1w", "2d") into Python timedelta objects
- **Regex Pattern:** `((?P<years>\d+?)y)?((?P<months>\d+?)m)?((?P<weeks>\d+?)w)?((?P<days>\d+?)d)?((?P<hours>\d+?)h)?`
- **Note:** Has a bug - doesn't multiply parsed values, always adds 1 for each unit type present

#### `repostorefresh() -> (list, list)`
- **Location:** git-update:52-83
- **Purpose:** Reads config files and determines which repos need updating
- **Returns:** Tuple of (repos_to_update, already_run_repos)

#### `rerun(repo: dict) -> bool`
- **Location:** git-update:86-89
- **Purpose:** Determines if a repository should be updated based on last run time and frequency
- **Logic:** Returns True if never run or if `lastRun + frequency < now()`

#### `update(repo: str) -> bool`
- **Location:** git-update:91-99
- **Purpose:** Performs the actual git pull operation
- **Returns:** True if successful, False if skipped (bare or dirty repo)
- **Side Effects:** Prints warning message if repo is dirty

## Code Conventions & Patterns

### Style Guidelines

1. **Python 3 Syntax:** This codebase uses Python 3
   - Use `print()` as a function with parentheses
   - String formatting uses `.format()` method
   - Type hints are not currently used but can be added if desired

2. **Naming Conventions:**
   - Functions: `lowercase` or `camelCase` (inconsistent - uses both)
   - Variables: `camelCase` (e.g., `allRepos`, `lastRun`)
   - Dictionary keys: `camelCase` (e.g., `lastRun`, `frequency`)

3. **Code Structure:**
   - Function definitions first
   - Main execution logic at module level (not wrapped in `if __name__ == '__main__'`)
   - Minimal error handling - uses basic conditionals rather than try/except

4. **Comments:**
   - File header comment explains configuration files
   - Inline comments are minimal
   - Function-level comments are sparse

### Important Implementation Details

1. **GitPython Usage:**
   - Uses `git.Repo(path)` to initialize repository objects
   - Uses `.remote().pull()` for updates (assumes default remote 'origin')
   - No error handling for git operations

2. **File Operations:**
   - Uses `os.path.expanduser()` for tilde expansion
   - Uses context managers (`with` statements) for file operations
   - Tab-delimited file format

3. **Time Handling:**
   - Uses `datetime.now()` for current time
   - Stores timestamps as Unix epoch with microseconds using `strftime('%s.%f')`
   - Uses `datetime.fromtimestamp()` to parse timestamps

## Known Issues & Bugs

### Bug in `parsefrequency()` (git-update:31-49)

The function has a logic error in parsing frequency values:

```python
if "weeks" in parts:
    weeks = weeks + 1  # Should be: weeks + int(parts["weeks"])
```

All unit types add a fixed value of 1 instead of using the parsed numeric value. This means:
- "1w" and "99w" both result in 1 week
- "2d" and "50d" both result in 1 day

**Impact:** Medium - frequency parsing doesn't work as documented, but pattern is still usable with single-digit values of 1.

### Missing Error Handling

1. **Git Operations:** No try/except for git pull failures (network issues, conflicts, etc.)
2. **File I/O:** No handling for missing or malformed config files
3. **Path Expansion:** No validation that repository paths exist

### Python 3 Migration Complete

The script has been migrated to Python 3 (`#!/usr/bin/env python3`). All functionality is compatible with modern Python versions.

## Development Guidelines for AI Assistants

### When Making Changes

1. **Maintain Python 3 Compatibility:**
   - Use Python 3 syntax and features
   - Python 3-only features (type hints, f-strings, etc.) are acceptable
   - Test changes work with Python 3.x

2. **Maintain Backward Compatibility:**
   - Don't change the format of configuration files without migration path
   - Existing `~/.update.repos` and `~/.update.lastruns` files must continue to work

3. **Follow Existing Patterns:**
   - Use camelCase for new variables to match existing style
   - Use tab delimiters in file formats
   - Keep functions simple and focused

4. **Testing Considerations:**
   - This repository has no test suite
   - Manual testing requires creating test config files
   - Consider git repository state (dirty/clean) when testing

### Common Development Tasks

#### Bug Fixes

**Fixing the parsefrequency bug:**
```python
# Current (broken):
if "weeks" in parts:
    weeks = weeks + 1

# Fixed:
if "weeks" in parts and parts["weeks"]:
    weeks = weeks + int(parts["weeks"])
```

#### Future Enhancements

Potential improvements for Python 3:
1. Add type hints for better code documentation
2. Consider using f-strings for string formatting
3. Add more robust error handling with specific exception types
4. Consider using pathlib for path operations
5. Add logging instead of print statements

#### Adding Error Handling

When adding error handling:
1. Wrap git operations in try/except blocks
2. Catch `git.exc.GitCommandError` for git-specific errors
3. Print user-friendly error messages
4. Continue processing remaining repos even if one fails

#### Adding Features

Common feature requests might include:
- Support for multiple remotes
- Support for specific branches
- Dry-run mode
- Verbose output option
- Email notifications on errors

When adding features:
- Add command-line argument parsing (use `argparse`)
- Maintain backward compatibility (no args = current behavior)
- Update README.md with new usage instructions

### File Modification Guidelines

#### Editing `git-update`

- **Read first:** Always read the current file before making changes
- **Preserve header:** Keep the copyright and license header intact
- **Test imports:** Ensure all imported modules are available
- **Maintain flow:** Keep the linear execution flow at module level

#### Updating `README.md`

- Keep the simple, concise style
- Update usage examples if behavior changes
- Document new configuration options clearly
- Include migration notes for breaking changes

### Git Workflow

This repository uses a simple workflow:

1. **Current Branch:** `claude/add-claude-documentation-tA59K`
2. **Commit Messages:** Follow the existing style (simple, descriptive)
3. **Pushing:** Use `git push -u origin <branch-name>` for new branches

**Example commit message:**
```
Fix parsefrequency bug to use actual numeric values

The parsefrequency function was adding 1 for each time unit instead
of using the parsed numeric value, causing "2w" and "99w" to both
be interpreted as 1 week.
```

## Dependencies

### Runtime Dependencies

- **GitPython** (`git` module): Git repository manipulation
  - Not in standard library, must be installed via pip
  - Installation: `pip3 install -r requirements.txt`
  - Minimum version: 3.1.0

### System Requirements

- Python 3.x (tested with Python 3.6+)
- Git installed and available in PATH
- Unix-like environment (uses `~` expansion, designed for cron)

## Important Notes for AI Assistants

1. **Configuration Files Are External:** The script reads user configuration from `~/.update.repos` and `~/.update.lastruns`. These files are NOT part of the repository.

2. **No Tests:** There are no automated tests. When making changes, explain how to manually test the functionality.

3. **Simple Scope:** This is a single-purpose utility. Avoid over-engineering or adding unnecessary complexity.

4. **User Impact:** Changes to file formats or behavior could break existing user setups. Always consider backward compatibility.

5. **Error Messages:** When the script prints error messages (e.g., "is dirty"), these appear during automated runs. Make error messages clear and actionable.

6. **Silent Operation:** The script is designed to run quietly via cron. Avoid adding verbose output unless explicitly requested or errors occur.

7. **Main Branch:** The repository's main branch information is not currently available. Check with the user before creating pull requests.

## Quick Reference

### File Locations
- Main script: `/git-update`
- User config: `~/.update.repos` (not in repo)
- Last run data: `~/.update.lastruns` (not in repo)

### Key Imports
```python
import git            # GitPython - external dependency
import os             # Standard library
import re             # Standard library
from datetime import datetime, timedelta  # Standard library
```

### Frequency Format Examples
```
1w    = 1 week
2d    = 2 days
1w2d  = 1 week and 2 days (combined)
12h   = 12 hours
```

### Common Git Operations in Code
```python
gitrepo = git.Repo(path)           # Open repository
gitrepo.bare                        # Check if bare
gitrepo.is_dirty()                  # Check for uncommitted changes
gitrepo.remote().pull()             # Pull from default remote
```

---

**Last Updated:** 2025-12-24
**For:** AI Assistant (Claude) code analysis and development
