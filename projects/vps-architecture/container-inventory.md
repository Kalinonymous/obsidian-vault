# VPS Container Inventory — srv1016839
**Updated:** 2026-05-18
**Purpose:** Authoritative container reference for all agents

## Network Architecture
- `web` network: public services (172.21.0.x)
- `westgate_internal`: Westgate backend ↔ frontend only
- `claude_default`: Claude Code v2 only

## Container Inventory

### AGENCY CORE (YourNetworkPlug stack)

| Container | Image | IP | Ports | Status | Mounts | Notes |
|-----------|-------|-----|-------|--------|--------|-------|
| `hermes` | hermes-agent | 172.21.0.19 | — | ✅ Up 2h | `/home/deploy/.hermes → /opt/data`, source dirs ro | Primary AI agent |
| `hermes-dashboard` | hermes-agent | 172.21.0.20 | 9119 (int) | ✅ Up 11m | same mounts as hermes | Kanban board at ai-dashboard.westgategroupofschools.ke |
| `axe-executor` | 632ecc01be1d | 172.21.0.13 | 8000 (int) | ✅ Up 4h | `/home/deploy → /home/deploy`, docker.sock | Privileged VPS ops, REST API |
| `axe-dashboard` | 4bc589c8a895 | — | 8001 | ✅ Up 4h | none | Executor UI |
| `openclaw-gateway` | ghcr.io/openclaw/openclaw:latest | — | 18789-18790 | ✅ Up 1h (healthy) | `.openclaw → /home/node/.openclaw` | WhatsApp/Telegram/Discord bridge |

### AI/AGENTS

| Container | Image | IP | Ports | Status | Mounts | Notes |
|-----------|-------|-----|-------|--------|--------|-------|
| `claude-code-v2` | 47a7a7330ff1 | — | — | ✅ Up 10d | `.claude → /home/node/.claude` | Claude Code CLI on VPS |
| `openwebui` | c2e4723fdbca | — | 8080 | ✅ Up 10d (healthy) | `openwebui volume → /app/backend/data` | Open WebUI frontend |

### INFRASTRUCTURE

| Container | Image | IP | Ports | Status | Mounts | Notes |
|-----------|-------|-----|-------|--------|--------|-------|
| `traefik` | traefik:v3.6 | 172.21.0.18 | 80, 443 | ✅ Up 2d | `acme.json`, `dynamic_conf.yml`, docker.sock | Reverse proxy + SSL |
| `browserless` | 57d19e414d9f | — | 3000, 9222 | ✅ Up 5d | none | Chrome CDP browser automation |
| `wg-easy` | weejewel/wg-easy:latest | — | 51830-51831 | ✅ Up 10d | `wg-easy/data → /etc/wireguard` | WireGuard VPN (ai.westgategroupofschools.ke access) |
| `pihole` | pihole/pihole:latest | — | 53,67,80,443 | ✅ Up 5d (healthy) | `pihole/etc-dnsmasq.d`, `pihole/etc-pihole` | DNS + ad-blocking |

### SCHOOL (Westgate Group of Schools)

| Container | Image | IP | Ports | Status | Mounts | Notes |
|-----------|-------|-----|-------|--------|--------|-------|
| `school_main-app-1` | school_main-app:latest | — | 3008 | ✅ Up 10d | `app_uploads → /app/www/html/public/dashboard/uploads` | PHP school ops (live) |
| `school_main-db-1` | mysql:8.0 | — | 3306 (local) | ✅ Up 10d | `school_main_db_data → /var/lib/mysql` | School MySQL |

### WEBSITE (Westgate Group of Schools — production)

| Container | Image | IP | Ports | Status | Networks | Notes |
|-----------|-------|-----|-------|--------|----------|-------|
| `westgate-frontend` | westgate-frontend | — | 3000 | ✅ Up 10d | web, westgate_internal | Next.js frontend |
| `westgate-backend` | westgate-backend | — | 3001 | ✅ Up 10d | web, westgate_internal | Node.js API |

### ISP MANAGEMENT SYSTEM

| Container | Image | IP | Ports | Status | Mounts | Notes |
|-----------|-------|-----|-------|--------|--------|-------|
| `isp-frontend` | isp-management-system-frontend | — | 3004 | ✅ Up 10d | none | React frontend |
| `isp-api` | isp-management-system-api | — | 3005 | ✅ Up 10d | none | Node.js API |
| `isp-db` | postgres:15-alpine | — | 5432 | ✅ Up 10d (healthy) | `isp-db-data`, schema.sql | PostgreSQL |
| `isp-redis` | redis:7-alpine | — | 6379 | ✅ Up 10d (healthy) | `isp-redis-data → /data` | Redis cache |

### OTHER

| Container | Image | IP | Ports | Status | Mounts | Notes |
|-----------|-------|-----|-------|--------|--------|-------|
| `genieacs` | c7b42d018c14 | — | 3000,7547-7548,7557,7567 | ✅ Up 10d | none | TR-069 ACS for ISPs |
| `mongodb-host` | mongo:latest | — | 27017 | ✅ Up 3d | `mongodb_data → /data/db`, `mongodb_config → /data/configdb` | MongoDB |
| `plan-server` | nginx:alpine | — | 9876 | ✅ Up 46h | `.hermes/plans → /usr/share/nginx/html` | Plan server (internal) |

## Stopped/Removed Containers
- `approval-bridge` — **STOPPED** May 17 — was causing Telegram 409 conflict with Hermes

## Traefik Routes (ai-dashboard.westgategroupofschools.ke via vpn-only)

| Subdomain | Service | VPN-Only |
|-----------|---------|----------|
| westgategroupofschools.ke | westgate-frontend:3000 | No |
| www.westgategroupofschools.ke | westgate-frontend:3000 | No |
| api.westgategroupofschools.ke | westgate-backend:3001 | No |
| isp.westgategroupofschools.ke | isp-frontend:3000 | No |
| api-isp.westgategroupofschools.ke | isp-api:3000 | No |
| finance.westgategroupofschools.ke | school_main-app-1:80 | No |
| proxy.westgategroupofschools.ke | openclaw-gateway:18789 | No |
| acs.westgategroupofschools.ke | genieacs:3000 | No |
| api-acs.westgategroupofschools.ke | genieacs:7548 | No |
| pihole.westgategroupofschools.ke | pihole:80 | No |
| vpn.westgategroupofschools.ke | wg-easy:51821 | No |
| **ai.westgategroupofschools.ke** | openwebui:8080 | **Yes** |
| **ai-dashboard.westgategroupofschools.ke** | hermes-dashboard:9119 | **Yes** |

## Key Paths
- VPS host: `/home/deploy/`
- Hermes home: `/opt/data/` (maps to `/home/deploy/.hermes/` on host)
- OpenClaw workspace: `/home/node/.openclaw/workspace/` (maps to `/home/deploy/.openclaw/workspace/`)
- Executor: `http://172.21.0.1:8000` — key: `gbDKzzxdJ6uIE1Rij52lbQ6qyYLnk6xb9/lVZK6kBew=`