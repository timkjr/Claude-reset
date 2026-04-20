# AGENTS.md - Claude Session Keeper

This file provides guidelines for agentic coding agents operating in this repository.

## Project Overview

- **Name**: Claude Session Keeper (csk)
- **Type**: Node.js CLI tool (CommonJS)
- **Purpose**: Keeps Claude Code session limit warm by pinging before reset times
- **Entry Point**: `bin/csk` (executable)
- **Dependencies**: chalk, commander

## Commands

### Running the tool

```bash
# Basic usage
csk --help

# Start daemon
csk start

# Stop daemon
csk stop

# Check status
csk status

# Fire a ping immediately
csk ping now

# Show config
csk config show

# Set config (interactive)
csk config set target_time 17:00
csk config set days mon-fri

# Fetch reset times via OAuth API
csk reset-time

# View logs
csk logs -n 50
```

### Testing

This project has **no test suite**. Test manually:

```bash
csk ping now        # Test a ping
csk status          # Check status
csk reset-time      # Test OAuth API
```

### Linting

No linting tools are configured. Code should follow the style guidelines below.

## Code Style Guidelines

### General

- Language: JavaScript (Node.js, CommonJS with `require()`)
- No TypeScript
- Use Node.js 16+ compatible syntax
- Prefer ES2017+ features (template literals, async/await, etc.)

### Imports

- One `require()` per line
- Groupings (not required but recommended):
  1. Node built-ins (fs, path, child_process)
  2. External deps (chalk, commander)
  3. Local modules (`./paths`, `./config`)
- Example:
  ```javascript
  const fs = require('fs');
  const path = require('path');
  const { spawnSync } = require('child_process');
  const chalk = require('chalk');
  const { program } = require('commander');

  const { CONFIG_FILE, CSK_DIR } = require('./paths');
  const configLib = require('./config');
  ```

### Naming Conventions

- **Files**: snake_case (e.g., `ping.js`, `config.js`)
- **Functions/variables**: camelCase (e.g., `runPing`, `configFile`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `CONFIG_FILE`, `DEFAULTS`)
- **Config keys**: snake_case (e.g., `target_times`, `log_level`, `claude_path`)

### Error Handling

- Use try/catch blocks forsync operations
- Throw descriptive Error objects with messages
- Handle async errors with try/catch and proper logging
- Example:
  ```javascript
  try {
    const data = fs.readFileSync(CONFIG_FILE, 'utf8');
    // ...
  } catch {
    return { ...DEFAULTS };  // Return defaults on error
  }
  ```

### Async/Await

- Use async/await for asynchronous operations
- Handle errors with try/catch
- Avoid callback-style when possible

### Formatting

- Indentation: 2 spaces (no tabs)
- Semicolons: required
- Quotes: single quotes for strings (except when avoiding escaping)
- Trailing commas: allowed in multiline objects/arrays
- Line length: soft limit 100 chars
- Example:
  ```javascript
  function parseResetFromOutput(output) {
    const match = output.match(/resets\s+(\d{1,2}(?::\d{2})?(?:am|pm))\s*(?:\(([^)]+)\))?/i);
    if (!match) return null;
    // ...
  }
  ```

### File Structure

- `bin/csk` - CLI entry point and command definitions
- `lib/*.js` - Library modules (one module per file)
- Use `module.exports` with object export:
  ```javascript
  module.exports = { read, write, set, DEFAULTS };
  ```

### Console Output

- Use `chalk` for colored output
- Use `chalk.green()` for success, `chalk.red()` for errors, `chalk.yellow()` for warnings
- Use `chalk.dim()` for secondary/info text
- Example:
  ```javascript
  console.log(chalk.green('✓ Daemon started'));
  console.log(chalk.dim(`  Logs: ${LOG_FILE}`));
  console.error(chalk.red('Error:'), e.message);
  ```

### Configuration

- Config stored in JSON format at `~/.csk/config.json`
- State stored at `~/.csk/state.json`
- Use `config.js` lib to read/write config
- Default values defined in `lib/config.js` DEFAULTS object

### Module Patterns

- Keep modules focused (single responsibility)
- Pure functions where possible
- Avoid side effects in library modules
- CLI entry point (`bin/csk`) handles all side effects

### Debugging

- Daemon logs: `~/.csk/logs/daemon.log` (or via `journalctl` on Linux)
- Use `csk logs -n 50` to view
- Ping results logged to `~/.csk/logs/ping.log`

## Common Tasks

### Adding a new command

1. Add command in `bin/csk` using commander
2. Import required library modules
3. Use chalk for output formatting

### Adding config option

1. Add default in `lib/config.js` DEFAULTS object
2. Add handling in `lib/config.js` read/write/set functions
3. Use in relevant library modules

### Modifying ping logic

1. Edit `lib/ping.js`
2. Test with `csk ping now`
3. Check `~/.csk/logs/ping.log`

## Notes

- This is a simple Node.js project with no build step
- No automated tests - test manually with `csk ping now` and `csk status`
- Linux support is marked experimental
- The tool works on macOS (launchd) and Linux (systemd or direct spawn)