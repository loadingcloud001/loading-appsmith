# Appsmith Architecture

## Overview

Appsmith is an open-source low-code platform for rapidly building internal tools, admin panels, and dashboards. It connects to databases and APIs to create CRUD applications with drag-and-drop UI components.

## Deployment

```
+-------------------------------------------------------------+
|                  Nginx Reverse Proxy                         |
|                   (TLS-terminated)                           |
+-------------------------+-----------------------------------+
                          |
                          v
+-------------------------------------------------------------+
|                     Appsmith CE                              |
|                  (Docker Container)                          |
|                                                               |
|  +---------+  +---------+  +---------+  +---------+          |
|  |  Nginx  |  | Backend |  | MongoDB |  |  Redis  |          |
|  | (front) |  | (Java)  |  | (embed) |  | (embed) |          |
|  +---------+  +---------+  +---------+  +---------+          |
+-------------------------+-----------------------------------+
```

All components run inside a single container. Nginx serves the React frontend and proxies API requests to the Java backend on `localhost:8080`. Embedded MongoDB and Redis provide data and session storage — no external databases required.

## Ports

| Port | Service | Notes |
|------|---------|-------|
| 80 | Nginx (internal) | Frontend + API proxy |
| 8080 | Backend | Java Spring Boot server (internal only) |

Only port 80 is exposed internally; the reverse proxy handles TLS and external routing.

## Data Persistence

The named Docker volume `appsmith_stacks` is mounted at `/appsmith-stacks` inside the container. This preserves:

| Directory | Contents |
|-----------|----------|
| `/appsmith-stacks/data/mongodb` | All application data, pages, queries, w\idgets |
| `/appsmith-stacks/data/plugins` | Plugin configurations |
| `/appsmith-stacks/configuration` | App server settings |
| `/appsmith-stacks/certificate` | SSL certificates (not used — reverse proxy handles TLS) |

## Authentication

Appsmith uses built-in email/password authentication (not Auth0 SSO). Access control:

- **Signup disabled** (`APPSMITH_SIGNUP_DISABLED=true`) — new accounts created via admin invitation only
- **Admin emails** (`APPSMITH_ADMIN_EMAILS`) — designated superusers on first login
- **Internal tools access** — SSO not required; tools are standalone internal applications

## Environment Variables

| Variable | Description |
|----------|-------------|
| `APPSMITH_CUSTOM_DOMAIN` | Full HTTPS domain for redirects and URL generation |
| `APPSMITH_SIGNUP_DISABLED` | Disable public registration (`true` recommended) |
| `APPSMITH_ADMIN_EMAILS` | Comma-separated list of superuser emails |
| `APPSMITH_ENCRYPTION_PASSWORD` | Encryption key for sensitive data |
| `APPSMITH_ENCRYPTION_SALT` | Encryption salt for sensitive data |

## Dependencies

```
Appsmith — standalone, no external services needed
    ├── Internal: MongoDB (embedded)
    ├── Internal: Redis (embedded)
    └── External: Nginx reverse proxy (TLS termination)
```

## Connection to NGINX Reverse Proxy

```
nginx.conf server block:
  server_name: appsmith.loadingtechnology.app
  proxy_pass:  http://loading-appsmith:80
```

All traffic goes through the reverse proxy which terminates TLS and forwards to the internal nginx on port 80.

## Backup

- **Application data**: Stored in MongoDB inside the `appsmith_stacks` Docker volume
- **Backup strategy**:
  1. Docker volume backup: `docker run --rm -v appsmith_stacks:/data -v $(pwd):/backup alpine tar czf /backup/appsmith-backup.tar.gz -C /data .`
  2. Or export apps via Appsmith's built-in JSON export feature
- **Frequency**: Weekly volume backup recommended

## Upgrade

Appsmith upgrades follow the standard service pattern:

```bash
cd /opt/loading-appsmith
docker compose pull          # Get latest image
docker compose down          # Stop container
docker compose up -d         # Start with new image
# Embedded MongoDB handles schema migrations automatically
```

For major version bumps, take a volume backup first.

## Technical Notes

- Appsmith's embedded MongoDB uses WiredTiger storage engine — adequate for small-to-medium deployments
- For high-traffic production use, consider external MongoDB (not implemented — Appsmith CE supports external MongoDB via `APPSMITH_MONGODB_URI`)
- The container supervisor manages all internal processes (nginx, backend, MongoDB, Redis)
- First boot takes ~2 minutes as embedded MongoDB initializes
