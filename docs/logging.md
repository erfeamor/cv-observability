# Logging pipeline

Deliberately separate from the metrics stack (Prometheus/Grafana).

## What ships today (T-054, 2026-10-08)

In production, the `cv-domain-service` and `cv-bff-node` containers' stdout and stderr go to **CloudWatch Logs** through Docker's `awslogs` driver on the app host:

| Group | Stream | Retention |
|---|---|---|
| `/cv-project/cv-domain-service` | `domain-service-<instance id>` | 14 days |
| `/cv-project/cv-bff-node` | `bff-node-<instance id>` | 14 days |

- The driver runs in non-blocking mode with a 4 MB buffer, so an app never stalls on logging. During a long CloudWatch outage the **newest** lines are dropped, and if the IAM grant were missing nothing would arrive at all. `docker logs` on the host keeps working either way.
- The domain service's events are split on its ISO timestamps, so a Java stack trace stays one event.
- MySQL's logs stay on the host. The CI host's doorbell and reaper Lambdas log to their own groups.

Read them with:

```bash
aws logs tail /cv-project/cv-domain-service --follow --region eu-west-3
aws logs tail /cv-project/cv-bff-node --follow --region eu-west-3
```

The infrastructure side (driver options, the IAM grant, the runbook) is in `cv-infra` (`templates/domain-service-provision.sh`, `docs/runbooks/app-host-deploy.md`).

## Out of scope (T-052)

The original design shipped **structured JSON** logs to MongoDB Atlas (free tier) or CloudWatch. T-052 decided not to build it: the apps log plain text, and Atlas is not deployed. If it is ever built:

- Java: Logback's JSON encoder (`logstash-logback-encoder`) on stdout. The `awslogs` driver above would carry it unchanged, so no appender is needed for CloudWatch.
- Node: `pino` on stdout, matching the Java log shape.
- Atlas would need its own sink (`pino-mongodb` or a Mongo appender) and credentials. That is a separate decision, with a cost.
