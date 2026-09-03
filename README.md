# Self-hosted n8n (Windows, CPU Ollama)

Personal Docker Compose stack: [n8n](https://n8n.io/) + [Ollama](https://ollama.com/) (CPU) + [Qdrant](https://qdrant.tech/) + [PostgreSQL](https://www.postgresql.org/).

This is a localhost kit for a Windows machine with **no dedicated GPU**. CPU inference is slow; stick to a small model (`llama3.2`, ~3B). Larger models will be very slow and RAM-heavy.

## Requirements

- [Docker Desktop for Windows](https://docs.docker.com/desktop/setup/install/windows-install/) with the **WSL2** backend
- **≥8 GB RAM** assigned to Docker Desktop (Settings → Resources) so n8n, Postgres, Qdrant, and a 3B model can coexist

## Setup

```powershell
copy .env.example .env
```

Replace `change-me` in `.env` for `POSTGRES_PASSWORD`, `N8N_ENCRYPTION_KEY`, and `N8N_USER_MANAGEMENT_JWT_SECRET`. Generate values with Git Bash / WSL:

```bash
openssl rand -hex 32
```

or PowerShell:

```powershell
-join ((1..32) | ForEach-Object { '{0:x2}' -f (Get-Random -Maximum 256) })
```

Set `GENERIC_TIMEZONE` and `TZ` to your Windows timezone if it is not `Europe/London`.

**Do not change `N8N_ENCRYPTION_KEY` after the first successful n8n start** — it unlocks stored credentials.

```powershell
docker compose up -d
```

Ollama starts with the stack (no `--profile`). A one-shot container pulls `llama3.2`; n8n may come up before that download finishes.

1. Open <http://localhost:5678> and create the owner account immediately (setup is open until you do).
2. In n8n: **Credentials** → add Ollama with base URL `http://ollama:11434`.
3. For RAG workflows, add Qdrant with URL `http://qdrant:6333` (Qdrant is not published to the host).

n8n is bound to `127.0.0.1:5678`. Ollama is on `127.0.0.1:11434` for local tools. Qdrant is internal-only.

## Shared files

`./shared` is mounted into n8n at `/data/shared`. Use that path in Read/Write Files, Local File Trigger, and Execute Command nodes.

## Data and upgrades

Named volumes hold the real data: `n8n_storage`, `postgres_storage`, `qdrant_storage`, `ollama_storage`. `docker compose down -v` deletes them.

```powershell
docker compose pull
docker compose up -d
```

Pinned images: `n8nio/n8n:2.37.7`, `qdrant/qdrant:v1.19.0`, `postgres:16-alpine`.

## License

Apache License 2.0 — see [LICENSE](LICENSE).

## Support

[n8n Forum](https://community.n8n.io/)
