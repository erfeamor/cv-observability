# Logging pipeline

Deliberately separate from the metrics stack above (Prometheus/Grafana). Structured JSON logs from `cv-domain-service` and `cv-bff-node` ship to one of:

- **MongoDB Atlas (free tier)** — for demoing a NoSQL sink and ad-hoc querying of events/audit trail.
- **CloudWatch Logs** — the simpler option when everything already runs on AWS.

Neither is wired up yet. When implemented:

- Java: use Logback's JSON encoder (`logstash-logback-encoder`) and either the MongoDB appender or the CloudWatch Logs SDK appender.
- Node: use `pino` with `pino-mongodb` or the CloudWatch transport, matching whichever sink the Java side uses so log shape stays consistent across services.
