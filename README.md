# Expense Tracker MCP Server

A lightweight MCP server for tracking personal expenses, built with FastMCP, Python 3.13, and SQLite (`aiosqlite` at runtime).

## Features

- Add expense entries with date, amount, category, optional subcategory, and note
- List expenses within a date range
- Summarize expenses by category for a date range
- Expose category data through an MCP resource
- Use async SQLite operations for runtime DB access

## Tech Stack

- Python 3.13
- [FastMCP](https://github.com/jlowin/fastmcp)
- `aiosqlite`
- `uv` for dependency and environment management

## Project Structure

- `main.py` – MCP server, tools, resource, and startup logic
- `categories.json` – category/subcategory data source for the categories resource
- `pyproject.toml` – project metadata and dependencies

## Setup

1. Install dependencies:

   ```bash
   uv sync
   ```

2. Run the server:

   ```bash
   uv run python main.py
   ```

The server starts on HTTP transport at `0.0.0.0:8000`.

## MCP Tools

### `add_expense(date, amount, category, subcategory="", note="")`

Inserts a new expense row into the `expenses` table.

Returns:
- Success: `{"status": "success", "id": <expense_id>, "message": "..."}`
- Error: `{"status": "error", "message": "..."}`

### `list_expenses(start_date, end_date)`

Returns all expenses where `date` is between the provided start and end dates (inclusive), ordered by latest first.

### `summarize(start_date, end_date, category=None)`

Returns grouped totals and counts by category for the selected date range.  
If `category` is provided, summary is filtered to that category.

## MCP Resource

### `expense:///categories` (`application/json`)

Returns category data from `categories.json`.  
If the file is missing, a default category list is returned.

## Database

- DB file path: system temp directory (`expenses.db`)
- Journal mode: WAL
- Table schema:
  - `id` (INTEGER PRIMARY KEY AUTOINCREMENT)
  - `date` (TEXT, required)
  - `amount` (REAL, required)
  - `category` (TEXT, required)
  - `subcategory` (TEXT, optional)
  - `note` (TEXT, optional)

## Notes

- Database initialization happens when `main.py` is loaded.
- Runtime DB operations use `aiosqlite` with async/await patterns.
