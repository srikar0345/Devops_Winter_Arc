# Day 12 — Docker Compose

## Objective
Run a Flask application, PostgreSQL database, and Redis service using Docker Compose.

## Architecture
- web: Flask application exposed on host port 5000
- db: PostgreSQL 16 with named volume pgdata
- redis: Redis 7 for request-hit counting
- Compose provides service-name DNS and a shared network

## Start the application
```bash
docker compose config -q
docker compose up -d --build
docker compose ps
```

## Test the endpoints
```bash
curl http://localhost:5000/health
curl http://localhost:5000/
curl http://localhost:5000/
```

The health endpoint returns `ok`. The homepage displays a Redis hit count and PostgreSQL visit count.

## Persistence test
1. Request the homepage and record the PostgreSQL visit count.
2. Run `docker compose down` without `-v`.
3. Run `docker compose up -d`.
4. Wait for the services to become healthy.
5. Request the homepage again.

Observed: PostgreSQL visit count persisted across stack recreation. Redis hits reset because Redis has no persistent volume in this configuration.

## Health checks and dependencies
PostgreSQL uses `pg_isready`; Redis uses `redis-cli ping`. The web service uses `depends_on` with `condition: service_healthy`.

## Restart policy test
The web service is configured with `restart: unless-stopped`.
A controlled `docker kill` terminated the web container with exit code 137. It did not automatically restart in the observed test; `docker compose up -d web` restored it, and `/health` returned `ok`. Automatic restart remains unverified.

## Configuration and Git hygiene
- `.env` stores local credentials and must not be committed.
- `.env.example` documents required variables without the local password.
- `.gitignore` excludes `.env`, Python cache files, and virtual environments.

## Troubleshooting commands
```bash
docker compose ps
docker compose logs --tail=50 web
docker compose logs --tail=50 db
docker compose logs --tail=50 redis
docker inspect day5-compose-web-1 --format='{{json .State}}'
```

## Evidence checklist
- [ ] Screenshot of `docker compose config -q` succeeding
- [ ] Screenshot of `docker compose ps` with database and Redis healthy
- [ ] Screenshot of `/health` returning `ok`
- [ ] Screenshot of homepage requests showing both counters
- [ ] Screenshot showing PostgreSQL count persisted after `down` and `up`
- [ ] Screenshot/logs documenting the restart-policy test
- [ ] Architecture diagram saved as `architecture.png`
