# Health Check Lab

## Setup
```bash
docker compose up -d
```

## Verify Service
```bash
curl -i localhost:3000/health
curl -i localhost:3000/ready
curl localhost:3000/orders
```

`/health` is a shallow liveness check and does not use PostgreSQL. `/ready`
runs a database query and returns `503` when PostgreSQL is unavailable. Compose
polls `/ready`, so `docker compose ps` reports the API as unhealthy during a
database outage and healthy again after the database returns.

## Verify Recovery
```bash
docker compose stop db
curl -i localhost:3000/health
curl -i localhost:3000/ready
docker compose ps
docker compose start db
docker compose ps
```

The API's `restart: unless-stopped` policy restarts it if its process exits;
Docker Compose does not restart a running container solely because its health
status becomes unhealthy. See [CLOUD_RUN.md](CLOUD_RUN.md) for the cloud probe
mapping.
