# Novato Scan Documentation

Novato Scan is a modular toolchain for orchestrating vulnerability scans and consolidating the resulting intelligence into actionable reports. This document provides a high-level view of the project architecture, outlines local setup, and captures the most common usage patterns for contributors and operators.

## Architecture Overview

The codebase is organized around three primary layers that keep scanning logic, orchestration, and presentation concerns separated:

- **Scanner Adapters**: Adapters wrap third-party scanners (for example, Nmap or proprietary services) and normalize their output into a common schema. They expose a single `scan(target, options)` entrypoint so that they can be swapped without touching the rest of the system.
- **Pipeline Orchestrator**: The orchestrator coordinates concurrent scans, manages scheduling, and handles retry logic. It owns the central task queue and enforces rate limiting and credential hygiene for each target environment.
- **Reporting Layer**: Once scan data is normalized, the reporting layer enriches the findings, persists them to storage, and emits human-friendly summaries (HTML, CSV, or API payloads) for downstream consumers.

Supporting packages—such as configuration loaders, shared logging utilities, and persistence drivers—live alongside these layers under domain-specific directories (for example `config/`, `logging/`, and `storage/`). Integration surfaces such as REST endpoints or CLI commands are thin wrappers that delegate to the orchestrator and reporting services.

## Local Setup

1. **Prerequisites**
   - Python 3.11+
   - Poetry for dependency management
   - Docker (optional) for containerized scanner adapters

2. **Installation**
   ```bash
   poetry install
   ```

3. **Environment Configuration**
   - Copy `.env.example` to `.env` and update secrets for upstream scanners.
   - Configure credentials for any authenticated adapters (for example, cloud APIs) before running scans.

4. **Database & Services**
   - Launch supporting services with `docker compose up -d` when adapters require auxiliary databases or message brokers.

## Usage

### Command Line Interface

Run ad-hoc scans through the CLI once dependencies are installed:

```bash
poetry run novato-scan scan --target 10.0.0.0/24 --profile full
```

Key options:

- `--profile`: Chooses a predefined collection of adapters to execute (for example `full`, `lightweight`, or `web-only`).
- `--output`: Overrides the default report destination (JSON, HTML, or CSV).
- `--resume`: Continues a paused pipeline using saved state.

### API Server

For continuous integrations, start the API server and trigger scans programmatically:

```bash
poetry run novato-scan api --port 8080
```

Use the `/scans` endpoint to submit jobs, poll `/scans/{id}` for status, and fetch reports from `/scans/{id}/report`.

### Development Workflow

- Run `poetry run pytest` to execute the automated test suite.
- Format code with `poetry run black .` and lint with `poetry run ruff check .` before sending pull requests.
- Use feature branches and keep commits focused on a single change whenever possible.

## Troubleshooting

- Verify network reachability when adapters fail—rate limiting or firewall rules are common culprits.
- Inspect orchestrator logs in `logs/` for scheduling or queue-related issues.
- Ensure Docker services are healthy if containerized adapters are part of the selected profile.

For additional questions, open an issue in the repository or contact the maintainers listed in `CODEOWNERS`.
