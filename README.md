# Docker Compose Assignment

This project demonstrates how to build a multi-service web application using Docker Compose. Through 6 progressive challenges, I learned how to containerize a full-stack application with dependencies, data persistence, health checks, and auto-restart capabilities.

## Architecture

The application consists of 4 services:

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Frontend   │────▶│   Backend   │────▶│  Database   │
│  (Nginx)    │     │   (Nginx)   │     │  (Postgres) │
└─────────────┘     └─────────────┘     └─────────────┘
                           │
                           │    ┌─────────────┐
                           └────▶│    Redis    │
                                 │   (Cache)   │
                                 └─────────────┘
```

## Services

| Service   | Image       | Port | Description                    |
|-----------|-------------|------|--------------------------------|
| frontend  | nginx:latest| 3000 | User interface                 |
| backend   | nginx:latest| 8080 | API backend                   |
| database  | postgres:15 | 5432 | PostgreSQL database            |
| redis     | redis:7     | 6379 | Redis cache                   |

## What I Learned

### Challenge 1: Launch the Basic App
- **Learned**: How to define multiple services in Docker Compose
- **Key Concept**: Using `depends_on` to control service startup order
- The frontend waits for backend to start before launching

### Challenge 2: Add a Database
- **Learned**: How to add PostgreSQL as a service with environment variables
- **Key Concept**: Setting `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`
- Using `depends_on` to ensure backend starts after database

### Challenge 3: Don't Lose the Data
- **Learned**: How to persist data using Docker volumes
- **Key Concept**: Named volumes (`pgdata`) store database data outside containers
- Data survives `docker compose down` and `docker compose up`

### Challenge 4: Wait Until DB is Actually Ready
- **Learned**: Container running ≠ service ready
- **Key Concept**: Health checks with `pg_isready` verify PostgreSQL is accepting connections
- Backend waits for database to be *healthy*, not just running

### Challenge 5: Add Redis for Caching
- **Learned**: How to add Redis with health checks
- **Key Concept**: Redis health check uses `redis-cli ping`
- Backend waits for both database and redis to be healthy

### Challenge 6: Survive a Docker Restart
- **Learned**: Containers don't auto-restart when Docker daemon restarts
- **Key Concept**: `restart: unless-stopped` policy makes services resilient
- All services automatically restart after `systemctl restart docker`

## Key Docker Compose Concepts

### Health Checks
```yaml
healthcheck:
  test: ["CMD", "pg_isready", "-U", "admin", "-d", "mydb"]
  interval: 5s
  timeout: 5s
  retries: 5
```

### Service Dependencies with Health Checks
```yaml
depends_on:
  database:
    condition: service_healthy
```

### Data Persistence
```yaml
volumes:
  - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

### Auto-Restart Policy
```yaml
restart: unless-stopped
```

## Commands

```bash
# Start all services
docker compose up -d

# Check service status
docker compose ps

# View logs
docker compose logs -f

# Stop all services
docker compose down

# Restart a specific service
docker compose restart backend
```

## Verification

- Data persists after container restart
- All services auto-restart after Docker daemon restart
- Health checks ensure proper service startup order