# TerseContext

An end-to-end pipeline from feature request to working code on `main`.

```
user request → breakdown → fracture → beads → whittler → working code on main
```

A human submits a feature request. **Breakdown** researches it against your codebase. **Fracture** decomposes the approved work into dependency-ordered tasks. **Beads** tracks the tasks. **Whittler** picks them up, runs Claude Code agents in isolated containers, and lands the results. **TerseContext** is the code-intelligence layer that powers the research and decomposition steps.

## Repos

| Repo | What it does | Status |
|------|-------------|--------|
| [tersecontext](https://github.com/tersecontext/tersecontext) | Code indexing: parses repos into a knowledge graph (Neo4j + Qdrant), answers semantic queries about code. Powers Breakdown's research and Fracture's decomposition. | ✅ Static + query pipeline functional |
| [breakdown](https://github.com/tersecontext/breakdown) | Feature-request UI (web + Slack). Submit a request, get a research summary (affected files, complexity, risks), approve or reject. Approved tasks are published to a Redis stream. | ✅ Functional |
| [fracture](https://github.com/tersecontext/fracture) | MCP server. Reads an approved task, queries TerseContext for codebase context, decomposes it into dependency-ordered beads using an LLM, and creates the beads in `bd`. | ✅ Functional |
| [whittler](https://github.com/tersecontext/whittler) | Orchestrator. Polls `bd` for ready beads, claims them, runs Claude Code agents in Docker containers, validates the results, and merges to `main`. | ✅ Functional |

## Start Here

**New to the project?** See [CONTRIBUTING.md](./CONTRIBUTING.md) for system prerequisites and step-by-step setup.

**Just want to explore the code intelligence layer?** Start with [tersecontext](https://github.com/tersecontext/tersecontext) — it's self-contained:

```bash
cd tersecontext
cp .env.example .env   # add ANTHROPIC_API_KEY
make up
make demo              # see it working in ~5 minutes
```

**Want the full pipeline?** Start with TerseContext, then bring up Breakdown, Fracture, and Whittler in that order. See [CONTRIBUTING.md](./CONTRIBUTING.md) for the full startup sequence.

## The `bd` CLI (beads)

Both Fracture and Whittler depend on the `bd` CLI from [steveyegge/beads](https://github.com/steveyegge/beads):

```bash
go install github.com/steveyegge/beads/cmd/bd@latest
```

## Key Dependencies

| Dependency | Required by | Where to get it |
|------------|-------------|-----------------|
| Python 3.11+ | breakdown, fracture, whittler | python.org |
| Go | fracture (bd install), whittler (beads-mcp) | go.dev |
| Docker + Compose v2 | tersecontext, breakdown, whittler | docs.docker.com |
| `bd` CLI | fracture, whittler | `go install github.com/steveyegge/beads/cmd/bd@latest` |
| Anthropic API key | all | console.anthropic.com |
