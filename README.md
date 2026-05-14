# 🐳 MCP with Docker — 5-Minute Demo

A minimal, ready-to-run demo of the **Docker MCP Catalog & Toolkit** using the official `docker/mcp-gateway` image. Spin up a full MCP environment in under 2 minutes — no pip, no npm, no manual setup.

---

## What is MCP?

**Model Context Protocol (MCP)** is an open standard that allows AI agents (like Claude, Cursor, or any compatible client) to securely interact with external tools, APIs, and data sources through a unified interface.

Docker's MCP Catalog and Toolkit makes running MCP servers effortless:

- 🗂️ **MCP Catalog** — A centralized registry of trusted, containerized MCP servers (GitHub, DuckDuckGo, Puppeteer, Postgres, Slack, and more)
- 🛠️ **MCP Toolkit** — A control panel inside Docker Desktop to manage server lifecycles, credentials, and client connections
- 🔀 **MCP Gateway** — Aggregates all your enabled servers behind a single endpoint, so AI clients only need one configuration

---

## Prerequisites

| Requirement | Notes |
|---|---|
| [Docker Desktop](https://www.docker.com/products/docker-desktop/) (latest) | Enable **Beta features → Docker MCP Toolkit** in Settings |
| [Claude Desktop](https://claude.ai/download) | Or any MCP-compatible client (Cursor, VS Code, etc.) |

> **Before the demo:** Run `docker compose pull` to pre-cache the gateway image and avoid download time on stage.

---

## Project Structure

```
mcp-docker-toolkit/
├── docker-compose.yml         # Optional: run gateway as a persistent service
├── claude_desktop_config.json # Drop-in Claude Desktop configuration
└── README.md
```

---

## Quick Start

### 1. Connect Claude Desktop

Copy the config file to Claude Desktop's config directory:

```bash
cp claude_desktop_config.json \
  ~/Library/Application\ Support/Claude/claude_desktop_config.json
```

The config tells Claude Desktop to spawn the gateway container on demand via **stdio** — no HTTP server, no `npx`, no extra dependencies:

```json
{
  "mcpServers": {
    "docker-mcp-gateway": {
      "command": "docker",
      "args": [
        "run", "--rm", "-i", "--init",
        "-v", "/var/run/docker.sock:/var/run/docker.sock",
        "docker/mcp-gateway:latest",
        "--servers=duckduckgo,time"
      ]
    }
  }
}
```

> **How it works:** Claude Desktop launches `docker run` as a child process and communicates with the gateway over stdin/stdout (MCP stdio transport). The gateway then manages the individual MCP server containers internally.

Restart Claude Desktop.

### 2. Start Using Tools

Open a new chat in Claude Desktop and try:

```
Search DuckDuckGo for the Docker MCP Toolkit and give me a summary.
```

```
What time is it right now in Tokyo, New York, and São Paulo?
```

---

## How It Works

```
┌─────────────────┐     stdin/stdout      ┌──────────────────────────┐
│  Claude Desktop │ ────────────────────► │  docker/mcp-gateway      │
│  (MCP Client)   │   (MCP stdio/JSON-RPC)│  (child process via run) │
└─────────────────┘                       │  ┌────────────────────┐  │
                                          │  │  duckduckgo server │  │
                                          │  ├────────────────────┤  │
                                          │  │   time server      │  │
                                          │  └────────────────────┘  │
                                          └──────────────────────────┘
                                                      │
                                          Docker socket (host daemon)
```

Claude Desktop spawns the gateway as a child process (`docker run --rm -i`). The gateway speaks **MCP over stdio** and manages the individual server containers internally. No port binding or HTTP server needed.

---

## Adding More Servers

Edit `docker-compose.yml` and add servers to the `--servers` flag:

```yaml
command:
  - --servers=duckduckgo,time,github,puppeteer
```

Then apply the change:

```bash
docker compose up -d
```

Browse available servers in **Docker Desktop → MCP Toolkit → Catalog** or at the [Docker MCP Catalog](https://hub.docker.com/catalogs/mcp).

For servers that require authentication (e.g., GitHub), configure credentials in **Docker Desktop → MCP Toolkit → [server name] → Config**.

---

## Security

Each MCP server runs in an **isolated container** — no access to your host filesystem or network unless explicitly configured. This is one of the key advantages of the Docker-based approach over bare-metal MCP installs.

---

## Useful Commands

```bash
# View running containers
docker ps

# Follow gateway logs in real time
docker compose logs -f

# Restart the gateway
docker compose restart

# Pull latest images
docker compose pull
```

---

## Compatibility

| AI Client | Configuration |
|---|---|
| Claude Desktop | Copy `claude_desktop_config.json` to `~/Library/Application Support/Claude/` |
| Cursor | Add to `.cursor/mcp.json` with the same `url` field |
| VS Code (MCP extension) | Point to `http://localhost:8811/sse` |
| Custom agent | Any HTTP client that speaks MCP over SSE or Streamable HTTP |

---

## Resources

- [Docker MCP Toolkit Docs](https://docs.docker.com/ai/mcp-catalog-and-toolkit/)
- [Docker MCP Catalog on Docker Hub](https://hub.docker.com/catalogs/mcp)
- [Model Context Protocol Spec](https://modelcontextprotocol.io)
- [Claude Desktop](https://claude.ai/download)
