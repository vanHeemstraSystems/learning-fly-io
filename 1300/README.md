# 1300 — Monitoring, Metrics & Logs

## Fly Monitoring Overview

Fly.io provides built-in observability at multiple levels:
- **Logs** — stdout/stderr from your Machines
- **Metrics** — CPU, memory, network via Prometheus
- **Status** — Machine health and deploy history

---

## Logs

### Live-tail logs
```bash
fly logs                      # All machines, current app
fly logs -a my-app            # Specify app
fly logs --instance abc123    # Specific machine
fly logs -r ams               # Filter by region
```

### Log format
```
2024-01-15T10:23:45.123Z app[abc123] ams [info] Server started on port 8080
2024-01-15T10:23:46.456Z app[abc123] ams [info] GET /health 200 2ms
```

Fields:
- Timestamp (UTC)
- App name + Machine ID
- Region
- Log level (from your app)
- Message

### Best practices for structured logging
Use JSON logging so logs are parseable:

```javascript
// Node.js with pino
const pino = require('pino');
const log = pino();
log.info({ req_id: '123', path: '/api/users' }, 'Request received');
```

Output:
```json
{"level":30,"time":1705312024000,"req_id":"123","path":"/api/users","msg":"Request received"}
```

---

## Metrics (Prometheus)

Fly exposes a Prometheus metrics endpoint for each app:

```bash
fly dashboard metrics          # Open metrics dashboard in browser
```

### Built-in metrics available:
- `fly_app_cpu_seconds_total` — CPU usage
- `fly_app_memory_rss_bytes` — RSS memory
- `fly_app_connect_count` — Active connections
- `fly_app_request_count` — HTTP request count (if using http_service)
- `fly_machine_started_count` — Machine start events

### Expose custom metrics
Your app can expose a `/metrics` endpoint in Prometheus format, and Fly will scrape it:

```toml
[[metrics]]
  port = 9091
  path = "/metrics"
```

Node.js example with `prom-client`:
```javascript
const client = require('prom-client');
const register = new client.Registry();
client.collectDefaultMetrics({ register });

const httpRequests = new client.Counter({
  name: 'http_requests_total',
  help: 'Total HTTP requests',
  labelNames: ['method', 'status'],
  registers: [register]
});

app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
});
```

---

## Grafana (Built-in Dashboard)

Fly provides a hosted Grafana instance accessible via:

```bash
fly dashboard
```

Navigate to the Metrics tab for pre-built dashboards showing:
- Request rate
- Response times
- Error rates
- CPU / Memory trends

---

## Health Checks

Health check status is visible in `fly status`:

```bash
fly status
```

```
Machines
ID            PROCESS  VERSION  REGION  STATE    CHECKS   LAST UPDATED
abc123def456  app      3        ams     started  1 total  10s ago
```

Detail on checks:
```bash
fly checks list
fly checks list --app my-app
```

---

## Alerting

Fly does not have built-in alerting. For production apps, connect external monitoring:

- **Datadog** — use Fly's log drain + metrics endpoint
- **Sentry** — add Sentry SDK to your app
- **Uptime Robot / Better Uptime** — external HTTP monitoring
- **PagerDuty / OpsGenie** — via webhook from your monitoring tool

### Log drain (ship logs to external)
```bash
fly logs --json | your-log-shipper
```

Or configure via the Fly dashboard: **App → Monitoring → Log Shipping**.

---

## Tasks

### Task 1 — Tail live logs
1. Open two terminals.
2. Terminal 1: `fly logs`
3. Terminal 2: `curl https://my-app.fly.dev` a few times.
4. Observe the log output in Terminal 1.

### Task 2 — Add structured logging
1. Add JSON logging to your app (pino, winston, structlog, etc.).
2. Deploy.
3. Run `fly logs` and confirm you see JSON-formatted lines.

### Task 3 — Explore metrics dashboard
1. Run `fly dashboard`.
2. Navigate to the Metrics section.
3. Find CPU and memory graphs for your app.

---

## Next Steps

→ Continue to [`1400`](../1400/README.md) — Fly Sprites (AI Sandboxes)
