# n8n with PostgreSQL & Caddy (Local & Production)

This repository contains the Docker Compose configurations to run **n8n** with a dedicated **PostgreSQL** database:
- **Local Development**: Run on `localhost:5678` using [dev.docker-compose.yml](file:///c:/Users/Kushal/Desktop/n8n-local/dev.docker-compose.yml).
- **Production Deployment**: Run with **Caddy** (automatic SSL/TLS via Let's Encrypt) using [docker-compose.yml](file:///c:/Users/Kushal/Desktop/n8n-local/docker-compose.yml).

---

## 1. Local Development

To start the local stack without Caddy / SSL:

```powershell
docker compose -f dev.docker-compose.yml up -d
```

- **n8n Web UI**: [http://localhost:5678](http://localhost:5678)
- **PostgreSQL**: `localhost:5433` (mapped from container 5432 to avoid host port collisions)
- **Stop**: `docker compose -f dev.docker-compose.yml down`

---

## 2. Production Deployment (VPS)

### Prerequisites on your VPS
1. **Ubuntu Server** (e.g. 2GB RAM on Hetzner, DigitalOcean, Linode, AWS Lightsail, or Oracle Cloud).
2. **Docker & Docker Compose** installed:
   ```bash
   curl -fsSL https://get.docker.com | sh
   ```
3. **DNS Record**: Point an `A` record (e.g., `n8n.yourdomain.com`) to your server's public IP address.

### Deployment Steps
1. Clone this repository on the server:
   ```bash
   git clone https://github.com/kushalpok01/n8n-worklog.git
   cd n8n-worklog
   ```
2. Create and customize your production `.env` from [.env.example](file:///c:/Users/Kushal/Desktop/n8n-local/.env.example):
   ```bash
   cp .env.example .env
   nano .env
   ```
   *Set `DOMAIN_NAME=n8n.yourdomain.com`, generate a strong `POSTGRES_PASSWORD`, and generate a 32-byte `N8N_ENCRYPTION_KEY`.*
3. Start the production stack:
   ```bash
   docker compose up -d
   ```

Caddy will automatically request and renew a free Let's Encrypt SSL certificate for your domain.

---

## Helpful Commands

| Action | Local | Production |
|---|---|---|
| Start services | `docker compose -f dev.docker-compose.yml up -d` | `docker compose up -d` |
| View logs | `docker compose -f dev.docker-compose.yml logs -f` | `docker compose logs -f` |
| View n8n logs | `docker compose -f dev.docker-compose.yml logs -f n8n` | `docker compose logs -f n8n` |
| Stop services | `docker compose -f dev.docker-compose.yml stop` | `docker compose stop` |
| Take down services | `docker compose -f dev.docker-compose.yml down` | `docker compose down` |

---

## Data Persistence & Security

- **Data Volumes**:
  - `n8n_data`: Preserves all workflows, credentials, and execution history.
  - `postgres_data`: Persists all relational database tables.
  - `caddy_data` / `caddy_config`: Preserves SSL certificates.
  - `sandbox_tls`: Persists mTLS certificates for secure sandbox communication.
- **Production Database Security**: In production, PostgreSQL is bound strictly to `127.0.0.1` and is never exposed to the public internet.

---

## 3. n8n Code Sandbox (AI Assistant & Agents)

The stack includes the official **n8n Sandbox Service** (`n8n-sandbox-service`) for executing AI code securely in isolated containers:
- **`sandbox-certs`**: Automatically generates mTLS certificates on first launch.
- **`sandbox-api`**: Acts as the central coordination API (accessible internally at `http://sandbox-api:8080`).
- **`sandbox-runner-1`**: Runs sandboxed tasks using Docker-in-Docker.

If prompted in the UI:
- **Service URL**: `http://sandbox-api:8080` (or `http://localhost:8080` from host)
- **API Key**: Found in your `.env` under `SANDBOX_API_KEY`

