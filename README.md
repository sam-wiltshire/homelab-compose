# homelab-compose

Docker Compose definitions for my self-hosted homelab. Each service lives in its own
folder with its own `docker-compose.yml`; a single `.env` file at the repo root
supplies shared configuration (user IDs, host paths, API keys, and internal URLs)
to whichever services need it.

## Services

| Folder | Image | Default Port | Purpose |
|---|---|---|---|
| `radarr` | `lscr.io/linuxserver/radarr` | 7878 | Movie collection manager |
| `sonarr` | `lscr.io/linuxserver/sonarr` | 8989 | TV show collection manager |
| `prowlarr` | `lscr.io/linuxserver/prowlarr` | 9696 | Indexer manager for *arr apps |
| `qbittorrent` | `lscr.io/linuxserver/qbittorrent` | 8081 | Torrent client |
| `sabnzbd` | `lscr.io/linuxserver/sabnzbd` | 8080 | Usenet client |
| `autobrr` | `ghcr.io/autobrr/autobrr` | 7474 | Automated torrent filtering/announcer |
| `unpackerr` | `golift/unpackerr` | — | Extracts completed Radarr/Sonarr downloads |
| `audiobookshelf` | `ghcr.io/advplyr/audiobookshelf` | 13378 | Audiobook & podcast server |
| `gotify` | `gotify/server` | 8085 | Self-hosted push notifications |
| `nginx-proxy-manager` | `jc21/nginx-proxy-manager` | 80 / 443 / 81 | Reverse proxy with a web UI |
| `glance` | `glanceapp/glance` | 8080 | Dashboard aggregating the above services |

> **Note:** `sabnzbd` and `glance` both default to port 8080. If you run both, change
> one of them in its `docker-compose.yml` before starting the stack.

## Prerequisites

- Docker Engine
- Docker Compose v2 (the `docker compose` CLI plugin)

## Setup

1. Clone the repo.
2. Copy the example environment file and fill in your own values:
   ```bash
   cp .env.example .env
   ```
3. Edit `.env`:
   - `PUID` / `PGID` — output of `id $USER` on the host, so containers write files
     with your user's permissions.
   - `TZ` — your timezone (e.g. `Europe/London`).
   - Directory paths — where your media/downloads actually live on the host.
   - Per-service API keys/URLs — see below.
4. Start whichever services you want. Since each service has its own compose file,
   run commands from the repo root and point at both the compose file and the
   shared `.env`:
   ```bash
   docker compose --env-file .env -f radarr/docker-compose.yml up -d
   docker compose --env-file .env -f sonarr/docker-compose.yml up -d
   # ...repeat per service
   ```

## Environment variables (`.env`)

`.env` is gitignored and never committed — only `.env.example` (with placeholder
values) is tracked. Copy it and fill in real values for your network.

| Variable | Used by | Notes |
|---|---|---|
| `PUID`, `PGID`, `TZ` | most services | Standard LinuxServer.io image conventions |
| `AUDIOBOOKS`, `DOWNLOADS`, `MEDIA_ROOT`, `PODCASTS` | radarr, sonarr, qbittorrent, unpackerr, audiobookshelf, sabnzbd | Host paths bind-mounted into containers |
| `RADARR_API_KEY`, `RADARR_URL` | unpackerr, glance | API key from Radarr's Settings → General |
| `SONARR_API_KEY`, `SONARR_URL` | unpackerr, glance | API key from Sonarr's Settings → General |
| `PROWLARR_URL` | glance | |
| `QBITTORRENT_URL` | glance | |
| `AUTOBRR_URL` | glance | |
| `AUDIOBOOKSHELF_URL` | glance | |
| `GOTIFY_URL` | glance | |
| `NGINX_PROXY_MANAGER_URL` | glance | |
| `ROUTER_URL` | glance | Bookmark link only |
| `PROXMOX_NODE`, `PROXMOX_URL`, `PROXMOX_API_TOKEN` | glance | Token created under Proxmox Datacenter → Permissions → API Tokens |
| `TRUENAS_URL`, `TRUENAS_API_KEY` | glance | API key created under TrueNAS → My Profile → API Keys |
| `ADGUARD_URL`, `ADGUARD_USERNAME`, `ADGUARD_PASSWORD` | glance | AdGuard Home admin credentials |

The `*_URL` variables that feed `glance` are only used for its dashboard widgets
(status monitor, custom API cards, bookmarks) — set them to wherever each service
actually runs on your network. `glance/glance.yml` references them with
`${VARIABLE_NAME}` syntax, which glance resolves from the container's environment
at load time, so any new variable must also be added to `glance/docker-compose.yml`'s
`environment:` block to actually reach the container.

## Security

- `.env` (and any file matching `**/.env`) is excluded via `.gitignore`. Never commit
  real API keys, passwords, or tokens.
- Internal IPs in this repo are placeholders in `.env.example` — replace them with
  your own network's addresses in your local `.env`.
