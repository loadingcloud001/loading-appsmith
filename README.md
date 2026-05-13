# Loading Appsmith

Appsmith low-code platform deployment for the Loading Technology stack.

## Quick Start

```bash
# Copy environment template
cp .env.example .env
# Edit .env with your admin email and encryption keys

# Start the container
docker compose up -d

# Check health
docker compose ps
docker inspect loading-appsmith --format='{{.State.Health.Status}}'
```

## URL

[https://appsmith.loadingtechnology.app](https://appsmith.loadingtechnology.app)

## Deployment

Deployment is fully automated via GitHub Actions:

1. Push changes to `main` branch
2. CI pipeline validates `docker-compose.yml` and checks for leaked secrets
3. CD pipeline deploys to DigitalOcean droplet `157.245.148.63`
4. Health check confirms container is running

### Required GitHub Secrets

| Secret | Purpose |
|--------|---------|
| `SSH_PRIVATE_KEY` | SSH key for droplet access (shared across services) |
| `APPSMITH_ADMIN_EMAILS` | Admin email(s) for initial setup |
| `APPSMITH_ENCRYPTION_PASSWORD` | Encryption key (16 hex chars) |
| `APPSMITH_ENCRYPTION_SALT` | Encryption salt (16 hex chars) |

## Architecture

See [architecture.md](architecture.md) for detailed architecture documentation.

## Related Services

- [nginx-reverse-proxy](https://github.com/loadingcloud001/loading-nginx-reverse-proxy) — Routes traffic to Appsmith and other services
- [dev-standards](https://github.com/loadingcloud001/dev-standards) — Shared infrastructure conventions
