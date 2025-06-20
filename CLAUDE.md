# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

**📖 For comprehensive project information, see [README.md](README.md)**

## Project Overview

This is an ESPN Fantasy Baseball MCP (Model Context Protocol) Server that provides comprehensive access to ESPN Fantasy Baseball data through Claude Desktop and other MCP-compatible AI assistants. The server is built using FastMCP and the espn-api Python library.

## Development Commands

### Environment Setup
```bash
./setup.sh                    # Initial setup - creates .venv, installs dependencies, configures VS Code
```

### Server Management
```bash
./start-dev.sh               # Start the MCP server in stdio mode for local development
```

### Testing & Diagnostics
```bash
./start-dev.sh               # Primary development server - runs in stdio mode for easy testing
```

## Architecture

### Core Structure
- **Entry Point**: `baseball_mcp/baseball_mcp_server.py` - FastMCP server initialization and tool registration
- **Modular Design**: Each functionality area has its own module:
  - `auth.py` - ESPN authentication (ESPN_S2/SWID cookies)
  - `league.py` - League information, settings, standings, scoreboard
  - `roster.py` - Team rosters, info, schedules
  - `players.py` - Player stats, free agents, search
  - `transactions.py` - All transaction types (trades, waivers, add/drop)
  - `draft.py` - Draft results and analysis
  - `matchups.py` - Weekly matchups and boxscores
  - `metadata.py` - Position mappings, stat IDs, activity types
  - `utils.py` - Error handling, league caching, serialization

### Key Technical Details
- **Session-Based Auth**: ESPN credentials (ESPN_S2/SWID cookies) are stored in memory per session, not persisted
- **League Caching**: Leagues are cached by session_id + league_id to avoid repeated API calls
- **Error Handling**: Centralized error handling in `utils.py` with consistent error responses
- **ESPN API Patching**: `espn_api_patch.py` applies fixes for None cookies bug in the upstream library

### Tool Categories (40+ tools total)
1. **Authentication**: Store/clear ESPN credentials
2. **League Management**: Info, settings, standings, scoreboard
3. **Team Operations**: Rosters, schedules, team info
4. **Player Analysis**: Stats, free agents, top performers, search
5. **Transactions**: Recent activity, waivers, trades, add/drop, player history
6. **Draft Analysis**: Results, round picks, team picks, scarcity analysis
7. **Matchups**: Weekly results, detailed boxscores
8. **Metadata**: Position mappings, stat definitions, activity types

## Dependencies

- **Python**: 3.12+ (specified in pyproject.toml and setup scripts)
- **Core Libraries**: 
  - `espn-api>=0.45.0` - ESPN Fantasy API wrapper
  - `mcp[cli]>=1.5.0` - Model Context Protocol framework
- **Virtual Environment**: Created in `.venv/` by setup.sh

## Configuration

### Claude Desktop Integration
- Config file locations:
  - macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
  - Linux: `~/.config/claude-desktop/claude_desktop_config.json`
- Server appears as "espn-baseball" in Claude Desktop
- Uses `./start-dev.sh` as the command wrapper

### VS Code/Cursor Setup
- `.vscode/settings.json` configured by setup.sh
- Python interpreter points to `./.venv/bin/python3`
- Analysis paths include virtual environment site-packages

## Important Limitations

1. **Baseball Only**: This server is specifically for ESPN Fantasy Baseball (not football/basketball)
2. **Private League Access**: Requires ESPN_S2 and SWID cookies for private leagues
3. **Session-Based Auth**: Credentials don't persist across server restarts
4. **ESPN API Rate Limits**: Be mindful of rapid successive requests
5. **Current Year Default**: Uses current calendar year for baseball season

## Common Workflows

### Adding New Tools
1. Implement function in appropriate module (e.g., `players.py` for player-related tools)
2. Register tool in `baseball_mcp_server.py` using `@mcp.tool()` decorator
3. Follow existing patterns for error handling and response formatting
4. Test using `./start-dev.sh` for manual verification

### Authentication Troubleshooting
- Guide users to obtain ESPN_S2 and SWID cookies from browser developer tools
- Use `auth_store_credentials` tool to store credentials
- Re-authentication required after server restarts

### Error Handling Pattern
- All functions should use `handle_error()` from utils.py
- ESPN API errors often indicate authentication issues (401) or invalid IDs
- Consistent error response format across all tools