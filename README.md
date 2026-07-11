# cv-observability

Metrics and logs for the Currículum Interactivo project, kept as two deliberately separate pipelines.

Part of the [cv-project](../README.md) multi-repo. Pipeline: Jenkins or GitHub Actions.

## Metrics

- Prometheus scrapes `cv-domain-service` (`/actuator/prometheus`, via Micrometer) and `cv-bff-node` (`/metrics`, via prom-client).
- Grafana is provisioned with a Prometheus datasource out of the box.

```bash
docker compose up -d
# Prometheus: http://localhost:9090
# Grafana:    http://localhost:3001  (admin / admin)
```

## Logs

See [docs/logging.md](docs/logging.md) — structured JSON logs go to MongoDB Atlas or CloudWatch Logs, not to this Prometheus/Grafana stack.
