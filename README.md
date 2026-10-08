# cv-observability

Metrics and logs for the Currículum Interactivo project, kept as two deliberately separate pipelines.

Part of the [cv-project](../README.md) multi-repo. Pipeline: GitHub Actions (`.github/workflows/ci.yml` validates the compose file and the Prometheus config).

**This metrics stack is local-only by design** (decided at T-052, 2026-10-07): it runs in the dev stack, and nothing scrapes the deployed services. The edge returns 403 for `/metrics`.

## Metrics

- Prometheus scrapes `cv-domain-service` (`/actuator/prometheus`, via Micrometer) and `cv-bff-node` (`/metrics`, via prom-client).
- Grafana is provisioned with a Prometheus datasource out of the box.

```bash
docker compose up -d
# Prometheus: http://localhost:9090
# Grafana:    http://localhost:3001  (admin / admin)
```

## Logs

See [docs/logging.md](docs/logging.md). In production, the domain service's and the BFF's container output goes to CloudWatch Logs (T-054), not to this Prometheus/Grafana stack.
