# CLAUDE.md — cv-observability

Metrics stack for cv-project: Prometheus + Grafana via docker compose. **Deliberate architecture decision: metrics and logs are two separate pipelines** — this repo owns metrics; structured JSON logs go to MongoDB Atlas or CloudWatch (see `docs/logging.md`, not yet implemented). Don't unify them; the split is a stated design goal. Cross-repo context: meta repo CLAUDE.md one directory up.

## Commands

```bash
docker compose up -d       # Prometheus :9090, Grafana :3001 (admin/admin)
docker compose config -q   # validate compose changes
docker run --rm -v "$PWD/prometheus:/c:ro" --entrypoint promtool \
  prom/prometheus:v2.53.0 check config /c/prometheus.yml   # validate prom config
```

CI: `.github/workflows/ci.yml` runs exactly those two validations — run them locally before pushing.

## Layout & conventions

- `prometheus/prometheus.yml` — standalone scrape config: reaches services on the **host** via `host.docker.internal` (works on Linux only because of the `extra_hosts: host-gateway` mapping in the compose file — don't remove it). The meta repo's dev stack has its *own* prometheus config (`../devstack/prometheus.dev.yml`) using compose service names; changes to scrape targets usually belong in **both**.
- `grafana/provisioning/` — datasource + dashboard providers, mounted read-only. Dashboards go in `grafana/provisioning/dashboards/` as JSON next to `dashboards.yml` (none exist yet — backlog).
- Scrape endpoints by convention: Java exposes `/actuator/prometheus` (Micrometer), Node exposes `/metrics` (prom-client). New services follow one of those two shapes.
- Pin image versions (currently prometheus v2.53.0, grafana 11.1.0); no `:latest`.

## Code review guidance

Priorities, ranked:

1. **Scrape target changes made in only one place.** A change to `prometheus/prometheus.yml` targets that isn't mirrored in `../devstack/prometheus.dev.yml` (or a clear note why not) will silently break one of the two stacks.
2. **`host.docker.internal` / `extra_hosts` removed or altered** — this is what makes host-network scraping work on Linux; removing it breaks scraping silently (no error, just empty metrics).
3. **Unpinned image versions** (`:latest` or missing tag) on Prometheus/Grafana.
4. A new scrape target that doesn't follow the `/actuator/prometheus` (Java) or `/metrics` (Node) convention without a stated reason.

Don't flag:
- Logs and metrics staying as separate pipelines, or the absence of structured JSON logging — both are stated, deliberate scope boundaries for this repo (see `docs/logging.md`).
- No Grafana dashboard JSON yet — tracked as backlog, not a gap in an unrelated PR.

## Git workflow

`master` is protected — feature branch (`feat/…`) → push → PR via `gh`. Definition of done: both CI validations pass locally.
