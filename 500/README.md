# 500 — Configuration (`fly.toml`)

## What is `fly.toml`?

`fly.toml` is the **declarative configuration file** for your Fly App. It lives at the root of your project and tells Fly.io:
- How to build your app
- Which ports to expose
- Health check settings
- Scaling parameters
- Environment variables (non-secret)
- Volume mounts

---

## Minimal Example

```toml
# fly.toml
app = "my-fly-app"
primary_region = "ams"

[build]

[http_service]
  internal_port = 8080
  force_https = true
  auto_stop_machines = true
  auto_start_machines = true
  min_machines_running = 0
```

---

## Full Annotated Reference

```toml
# ─── App Identity ───────────────────────────────────────────────────────────
app = "my-fly-app"           # Unique app name within your org
primary_region = "ams"       # Default region for new Machines

# ─── Build ──────────────────────────────────────────────────────────────────
[build]
  # Auto-detected if Dockerfile present.
  # Or specify a builder:
  # builder = "paketobuildpacks/builder:base"
  # Or a prebuilt image:
  # image = "nginx:1.25"
  
  # Pass build args:
  [build.args]
    NODE_ENV = "production"

# ─── Environment Variables (non-secret) ──────────────────────────────────────
[env]
  LOG_LEVEL = "info"
  PORT = "8080"

# ─── HTTP Service (recommended for web apps) ─────────────────────────────────
[http_service]
  internal_port = 8080          # Port your app listens on inside the container
  force_https = true            # Redirect HTTP → HTTPS
  auto_stop_machines = true     # Stop idle Machines
  auto_start_machines = true    # Wake on incoming traffic
  min_machines_running = 0      # Minimum always-on Machines
  
  [http_service.concurrency]
    type = "requests"           # "requests" or "connections"
    soft_limit = 200            # Start new instances at this load
    hard_limit = 250            # Reject connections above this

# ─── TCP Services (alternative to http_service, for non-HTTP) ────────────────
# [[services]]
#   protocol = "tcp"
#   internal_port = 5432
#   [[services.ports]]
#     port = 5432

# ─── Health Checks ───────────────────────────────────────────────────────────
[[http_service.checks]]
  grace_period = "5s"           # Wait before first check after start
  interval = "10s"              # How often to check
  method = "GET"
  path = "/health"
  protocol = "http"
  timeout = "2s"
  tls_skip_verify = false

# ─── Mounts (Volumes) ────────────────────────────────────────────────────────
[[mounts]]
  source = "myapp_data"         # Volume name
  destination = "/data"         # Mount path inside container

# ─── VM Size ─────────────────────────────────────────────────────────────────
[[vm]]
  memory = "512mb"
  cpu_kind = "shared"
  cpus = 1

# ─── Processes ───────────────────────────────────────────────────────────────
[processes]
  app = "node server.js"
  worker = "node worker.js"

# ─── Statics (serve files from a directory) ──────────────────────────────────
[[statics]]
  guest_path = "/app/public"
  url_prefix = "/static"
```

---

## Applying Config

```bash
# Deploy with current fly.toml
fly deploy

# Validate fly.toml without deploying
fly config validate

# Show current deployed config
fly config show

# Pull remote config into local fly.toml
fly config save
```

---

## Environment Variables vs Secrets

| Use Case | Tool |
|----------|------|
| Non-sensitive config (`LOG_LEVEL`, `PORT`) | `[env]` in fly.toml |
| Sensitive values (`DATABASE_URL`, `API_KEY`) | `fly secrets set KEY=value` |

Secrets are **never** stored in fly.toml and are injected at runtime.

---

## Region Configuration

```toml
primary_region = "ams"   # Where new Machines land by default
```

To deploy to multiple regions, use the CLI:
```bash
fly scale count 1 --region lhr
fly scale count 1 --region syd
```

---

## Common Patterns

### Always-on web app (no cold starts)
```toml
[http_service]
  auto_stop_machines = false
  min_machines_running = 1
```

### Low-cost hobby app (sleep when idle)
```toml
[http_service]
  auto_stop_machines = true
  auto_start_machines = true
  min_machines_running = 0
```

### Background worker (no HTTP)
```toml
[processes]
  worker = "python worker.py"

# No [http_service] section needed
```

---

## Tasks

### Task 1 — Add a health check
1. Add a `/health` endpoint to your app returning `200 OK`.
2. Add the `[[http_service.checks]]` block to `fly.toml`.
3. Run `fly deploy` and verify health checks pass via `fly status`.

### Task 2 — Add environment variables
1. Add an `[env]` block with `APP_GREETING = "Hello from fly.toml"`.
2. Read it in your app code.
3. Deploy and verify.

### Task 3 — Validate your config
1. Intentionally introduce a typo in `fly.toml`.
2. Run `fly config validate` — observe the error.
3. Fix it.

---

## Next Steps

→ Continue to [`600`](../600/README.md) — Networking, Routing & Certificates
