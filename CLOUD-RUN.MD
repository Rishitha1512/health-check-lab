# Cloud Run Health Check Mapping

## Overview

This project adds two health endpoints and a Docker Compose health check for the `orders-api` service.

- `/health` is the **liveness** endpoint. It returns `200 OK` when the Node.js process is alive and does not depend on PostgreSQL.
- `/ready` is the **readiness** endpoint. It checks PostgreSQL using `SELECT 1`.
- `/ready` returns `200 READY` when the database is reachable.
- `/ready` returns `503 NOT READY` when the database is unavailable.
- Docker Compose polls `/health` and marks the container as `healthy` or `unhealthy`.
- The service uses `restart: unless-stopped`.

## Local Docker Mapping

| Local Implementation | Purpose |
|---|---|
| `GET /health` | Liveness check — confirms the application process is responding |
| `GET /ready` | Readiness check — confirms the application can reach PostgreSQL |
| Compose `healthcheck` | Allows Docker to monitor the application's health |
| `restart: unless-stopped` | Restart policy for the container if the process exits |

The Docker health check uses `/health` because liveness should remain a shallow check. A temporary database failure should not cause the liveness endpoint itself to fail.

## Cloud Run Mapping

The local health checks map conceptually to Google Cloud Run's startup, liveness, and readiness probes.

### Startup Probe

Cloud Run's startup probe determines whether the container has successfully started and is ready to begin normal operation.

The local `/health` endpoint can be used as the startup probe because it verifies that the HTTP server has started and can respond.

```yaml
startupProbe:
  httpGet:
    path: /health
    port: 3000
  periodSeconds: 10
  timeoutSeconds: 3
  failureThreshold: 3