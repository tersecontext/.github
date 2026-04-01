# Contributing to the TerseContext Ecosystem

## System Prerequisites

Install these before setting up any individual repo:

| Tool | Version | Install |
|------|---------|---------|
| Python | 3.11+ | [python.org](https://python.org) or `pyenv` |
| Go | 1.21+ | [go.dev](https://go.dev/dl/) |
| Docker | with Compose v2 | [docs.docker.com](https://docs.docker.com/get-docker/) |
| `bd` CLI | latest | `go install github.com/steveyegge/beads/cmd/bd@latest` |
| Anthropic API key | — | Set `ANTHROPIC_API_KEY` in environment or `.env` |

Optional (needed only for full TerseContext dynamic pipeline):
- Neo4j, Qdrant, Redis, Postgres — all started automatically by `docker compose up`

## Repo Overview

```
tersecontext/  — Code indexing service (start here)
breakdown/     — Feature-request UI → research summaries → Redis stream
fracture/      — MCP server: decomposes tasks into dependency-ordered beads
whittler/      — Orchestrator: claims beads, runs Claude Code agents, lands code on main
```

## Startup Order

Services must start in this order because each one depends on the previous:

```
1. tersecontext  →  make up && make demo   (starts Neo4j, Qdrant, Redis, Postgres, all pipeline services)
2. breakdown     →  docker compose up       (needs TerseContext running + repos indexed)
3. fracture      →  python src/fracture/server.py  (or consumer.py for Redis mode)
4. whittler      →  whittler run            (needs bd CLI + open beads + Docker)
```

You don't need to run all four at once. `tersecontext` alone is a self-contained code-indexing tool.

> **Note:** Breakdown requires TerseContext to have at least one repo indexed before it can research tasks. Run `make demo` (or index your own repo) before starting Breakdown.

## Setting Up Each Repo

### TerseContext

```bash
cd tersecontext
cp .env.example .env        # fill in ANTHROPIC_API_KEY and passwords
make up                     # start all services
make demo                   # verify with bundled sample repo (~5 minutes)
```

Indexing your own repo:
```bash
curl -X POST http://localhost:8091/install-hook \
  -d '{"repo_path": "/path/to/your/repo"}' \
  -H 'Content-Type: application/json'
```

### Breakdown

```bash
cd breakdown
docker network create tersecontext_tersecontext 2>/dev/null || true
cp .env.example .env        # set ANTHROPIC_API_KEY
docker compose up --build
# open http://localhost:5173
```

### Fracture

```bash
cd fracture
pip install -e .
cp fracture.yaml.example fracture.yaml 2>/dev/null || true  # or use fracture.yaml in project root
# Edit fracture.yaml and set tersecontext.endpoint to match your TerseContext port (default 8090):
#   tersecontext:
#     endpoint: "http://localhost:8090"
export ANTHROPIC_API_KEY=sk-ant-...
# MCP mode:
python src/fracture/server.py
# Redis consumer mode:
python src/fracture/consumer.py --config fracture.yaml
```

### Whittler

```bash
# 1. Install beads (provides bd CLI and beads-mcp):
go install github.com/steveyegge/beads/cmd/bd@latest

# 2. Find your beads-mcp path in the Go module cache:
BEADS_VERSION=v0.59.0
BEADS_MCP_PATH=$(go env GOPATH)/pkg/mod/github.com/steveyegge/beads@${BEADS_VERSION}/integrations/beads-mcp
# If go env GOPATH is empty, it defaults to ~/go

# 3. Update pyproject.toml to use that path, then install:
cd whittler
# Edit pyproject.toml: update the "beads-mcp @ file://..." line to use $BEADS_MCP_PATH
pip install -e .
cp whittler.yaml.example whittler.yaml  # edit as needed
docker build -t whittler-solver:latest docker/
export ANTHROPIC_API_KEY=sk-ant-...
whittler run
```

## Running Tests

Each repo has its own test suite:

```bash
# breakdown (requires running postgres + redis)
cd breakdown && docker compose up postgres redis -d && alembic upgrade head && pytest

# fracture
cd fracture && pytest

# whittler
cd whittler && pytest

# tersecontext
cd tersecontext && make test   # or: pytest src/
```

## Where to Get Help

- **Something not documented here?** Open an issue in the relevant repo.
- **bd CLI issues?** See [steveyegge/beads](https://github.com/steveyegge/beads).
- **TerseContext not indexing?** Check `docker compose logs repo-watcher` and `docker compose logs parser`.
