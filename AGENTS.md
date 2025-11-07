# Repository Guidelines

## Project Structure & Module Organization
- `jumpserver_mcp_server/`: Python package (FastAPI + MCP).
  - `server.py`: SSE MCP server bridging JumpServer OpenAPI -> tools.
  - `config.py`: Pydantic settings loaded from `.env`.
  - `__main__.py`: CLI entry; runs Uvicorn.
  - `setup.py`: logging config helper.
- `main.py`: repository entrypoint.
- `Dockerfile`: builds with `uv` (Python 3.11).
- `.github/workflows/build-image.yml`: image publish workflow.
- `pyproject.toml`/`uv.lock`: dependencies and Ruff lint config.

## Build, Test, and Development Commands
- Install deps: `uv sync`
- Run locally (reads `.env`): `uv run -m jumpserver_mcp_server` or `uv run main.py`
- Docker (dev): `docker build -t ghcr.io/jumpserver/mcp:dev .` then
  `docker run -d -p 8099:8099 --env-file .env --name jms_mcp ghcr.io/jumpserver/mcp:dev`
- Lint (dev tool): `uvx ruff check .`
- Tests (if added): `uvx pytest -q`

## Coding Style & Naming Conventions
- Python 3.11, 4-space indents, max line length 100 (`[tool.ruff]`).
- Use type hints and docstrings for public functions.
- Names: modules/functions `snake_case`; classes `CapWords`; constants `UPPER_SNAKE_CASE`.
- Lint with Ruff; optional formatter: `uvx ruff format .`.

## Testing Guidelines
- Framework: `pytest` (recommended). Place tests under `tests/` named `test_*.py`.
- Aim to cover API auth paths, OpenAPI fetch/convert, and SSE message flow.
- Example: `uvx pytest --cov=jumpserver_mcp_server -q` (if `pytest-cov` installed).

## Commit & Pull Request Guidelines
- Follow Conventional Commits (seen in history: `perf:`). Prefer `feat:`, `fix:`, `docs:`, `ci:`, `chore:`.
- PRs must include: concise description, linked issue, test notes, and any config changes.
- CI must pass and image must build: `docker build .`.

## Security & Configuration Tips
- Create `.env` (do not commit):
  ```
  server_port=8099
  jumpserver_url=http://<host>
  api_token=<JumpServerBearer>
  api_key=<optional inbound API key>
  base_path=/sse
  ```
- Keep `log_level=DEBUG` only in local dev. Protect tokens; rotate on leaks.
- Run behind TLS; the OpenAPI fetch uses `verify=False`; use trusted networks or proxy with TLS.
