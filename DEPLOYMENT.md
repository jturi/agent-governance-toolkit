# Local container deployment

This host runs the upstream Agent Governance Toolkit dashboard with the
server-specific Compose overlay in `docker-compose.server.yml`.

## Start

```bash
docker compose -p agent-governance-toolkit \
  -f docker-compose.yml -f docker-compose.server.yml \
  --profile dashboard up -d --build dashboard
```

The dashboard is available only on the host at:

```text
http://127.0.0.1:18501
```

It is intentionally not published on `0.0.0.0`. Put it behind an authenticated
reverse proxy or Tailscale access control before exposing it remotely. The
server overlay runs as the upstream non-root `dev` user, drops Linux
capabilities, uses a read-only root filesystem, and limits memory, CPU, and
process count.

## Operations

```bash
docker compose -p agent-governance-toolkit \
  -f docker-compose.yml -f docker-compose.server.yml \
  --profile dashboard ps

docker compose -p agent-governance-toolkit \
  -f docker-compose.yml -f docker-compose.server.yml \
  logs -f --tail 100 dashboard

docker compose -p agent-governance-toolkit \
  -f docker-compose.yml -f docker-compose.server.yml \
  --profile dashboard down
```

## Scope

The upstream toolkit is primarily a library and SDK. The container here runs
its official Streamlit hypervisor dashboard; application agents still need the
appropriate AGT SDK/plugin integration to enforce policies on their tool calls.
