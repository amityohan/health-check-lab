# Cloud Run Health Check Mapping

The local `GET /health` endpoint is a shallow liveness check: it confirms that
the Node.js process can answer HTTP and does not depend on PostgreSQL. The
`GET /ready` endpoint is a readiness check: it runs `SELECT 1` and returns
`503` while PostgreSQL is unavailable.

On Cloud Run, `/health` is suitable for liveness: a failed liveness probe can
restart an unresponsive instance. A startup probe should report that the
application has finished booting; it can use `/health` here because the server
is ready as soon as it begins accepting requests. Do not use `/ready` as
liveness or startup: a temporary database outage should not restart otherwise
healthy application instances. Cloud Run's built-in probes do not provide the
same dependency-aware readiness behavior as Kubernetes, so keep `/ready` for
application-level checks and observability. In Kubernetes, `/ready` maps to a
`readinessProbe` and `/health` maps to a `livenessProbe`.

The Compose healthcheck uses `/ready`, so stopping PostgreSQL makes the API
container report `unhealthy`, and restoring PostgreSQL makes it report
`healthy` again. `restart: unless-stopped` restarts the service after its
process exits or the Docker daemon restarts; Docker Compose does not restart a
container solely because its health status is `unhealthy`.