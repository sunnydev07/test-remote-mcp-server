# AGENTS.md - Guidelines for Agentic Coding Agents

## Project Overview
Expense Tracker MCP server built with FastMCP, Python 3.13, and async SQLite.

## Package Management
- **Manager**: `uv` (uses `.python-version` 3.13, `pyproject.toml`, `uv.lock`)
- **Install deps**: `uv sync`
- **Add dep**: `uv add <package>`
- **Run script**: `uv run <command>`
- **Run main**: `uv run python main.py`

## Commands
```bash
# Start server (HTTP transport on port 8000)
uv run python main.py

# Run with uv directly
uv run fastmcp dev main.py
```

## Testing
- No test framework is currently configured.
- If adding tests, use `pytest`: `uv add --dev pytest pytest-asyncio`
- Run all tests: `uv run pytest`
- Run single test: `uv run pytest tests/test_file.py::test_name -v`

## Linting/Formatting
- No linter/formatter is currently configured.
- Recommended: add `ruff` via `uv add --dev ruff`
- Format: `uv run ruff format .`
- Lint: `uv run ruff check .`
- Fix: `uv run ruff check --fix .`

## Code Style

### Imports
- Standard library imports first, then third-party, then local (grouped with blank lines)
- Use explicit imports: `from fastmcp import FastMCP`
- Inline imports inside functions only when needed (e.g., `import sqlite3` inside `init_db`)

### Formatting
- 4-space indentation (no tabs)
- Max line length: follow default (88 chars, like Black/ruff)
- Trailing commas in multi-line structures

### Types
- Use type hints on function signatures where practical
- Async functions must be declared with `async def`
- MCP tool functions are `async`; resources can be sync

### Naming Conventions
- Functions/variables: `snake_case` (e.g., `add_expense`, `DB_PATH`)
- Constants: `UPPER_SNAKE_CASE` (e.g., `DB_PATH`, `TEMP_DIR`)
- MCP server instance: lowercase (`mcp`)

### Error Handling
- Use try/except blocks around database operations
- Return structured error dicts: `{"status": "error", "message": "..."}`
- Return success dicts: `{"status": "success", ...}`
- Print diagnostics to stdout for initialization issues
- Re-raise critical errors during init (e.g., `init_db`)

### Async Patterns
- Use `aiosqlite` for all runtime DB operations (not sync `sqlite3`)
- Always `await` cursor operations: `await c.execute(...)`, `await cur.fetchall()`
- Use `async with aiosqlite.connect(...)` for connection management
- Sync `sqlite3` is acceptable only for one-time initialization

### MCP Patterns
- Tools: decorated with `@mcp.tool()`, must be `async def`
- Resources: decorated with `@mcp.resource("scheme:///path", mime_type="...")`
- Use triple-slash in resource URIs: `expense:///categories`
- Server transport: HTTP on `0.0.0.0:8000` (default), stdio if no args

### Database
- DB file stored in temp directory for write access: `tempfile.gettempdir()`
- Use WAL journal mode for better concurrency
- Schema: `expenses(id, date, amount, category, subcategory, note)`

## File Structure
```
main.py           # All MCP tools, resources, and server entry point
pyproject.toml    # Project metadata and dependencies
uv.lock           # Locked dependency versions
categories.json   # Optional custom categories (falls back to defaults)
```

## Notes
- No `.cursor/rules/`, `.cursorrules`, or `.github/copilot-instructions.md` exist
- Keep all logic in `main.py` unless the project grows significantly
- Commit messages: concise, imperative mood (e.g., "add expense summary tool")
