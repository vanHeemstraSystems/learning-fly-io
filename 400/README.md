# 400 — Fly Machines Deep Dive

## What is a Fly Machine?

A Fly Machine is a **hardware-virtualized container** — not just a Docker container, but a full KVM virtual machine running your OCI image. This provides:

- **True hardware isolation** (each Machine gets its own kernel)
- **~300ms cold start** (fast enough to boot per HTTP request)
- **Ephemeral or persistent** (with Volumes for state)
- **Per-second billing** (pay only when running)

---

## Machine Lifecycle

```
                  ┌─────────────────────────────────┐
                  │         Machine States           │
                  │                                  │
  fly machine ────►  created ──► starting ──► started │
  create           │                │               │  │
                  │           (health check)        │  │
                  │                ▼               ▼  │
                  │           stopping ◄── stopped    │
                  │                │                  │
                  │                ▼                  │
                  │           destroyed               │
                  └─────────────────────────────────┘
```

### States
- **created** — Machine is defined but not yet started
- **starting** — Machine VM is booting
- **started** — Machine is running and healthy
- **stopping** — Machine is shutting down
- **stopped** — Machine is idle (not billed for CPU)
- **destroyed** — Machine is deleted permanently

---

## Machine API (Machines REST API)

Fly exposes a REST API for direct Machine management, useful for programmatic scaling.

### Create a Machine

```bash
curl -X POST \
  "https://api.machines.dev/v1/apps/my-app/machines" \
  -H "Authorization: Bearer $(fly auth token)" \
  -H "Content-Type: application/json" \
  -d '{
    "config": {
      "image": "registry.fly.io/my-app:latest",
      "guest": {
        "cpu_kind": "shared",
        "cpus": 1,
        "memory_mb": 256
      },
      "services": [{
        "ports": [{"port": 443, "handlers": ["tls", "http"]}],
        "protocol": "tcp",
        "internal_port": 8080
      }]
    },
    "region": "ams"
  }'
```

### Start / Stop via CLI

```bash
fly machine start <machine-id>
fly machine stop <machine-id>
fly machine restart <machine-id>
```

---

## VM Sizes (Guest Config)

```bash
fly platform vm-sizes
```

Common sizes:

| Name | CPU | Memory | Notes |
|------|-----|--------|-------|
| `shared-cpu-1x` | 1 shared | 256 MB | Free tier eligible |
| `shared-cpu-2x` | 2 shared | 512 MB | |
| `performance-1x` | 1 dedicated | 2 GB | |
| `performance-2x` | 2 dedicated | 4 GB | |
| `performance-4x` | 4 dedicated | 8 GB | |

Change size:
```bash
fly scale vm performance-1x
fly scale memory 1024   # set to 1GB
```

---

## Process Groups

A single Fly App can run multiple **process groups** — different Machine configurations for different roles.

Example `fly.toml`:

```toml
[processes]
  app = "node server.js"
  worker = "node worker.js"

[[services]]
  processes = ["app"]
  internal_port = 8080
```

Scale independently:
```bash
fly scale count app=2 worker=1
```

---

## Machine Init & Entrypoint

You can override the Docker CMD/entrypoint per-Machine:

```bash
fly machine run registry.fly.io/my-app:latest \
  --entrypoint "/bin/sh" \
  -e MY_VAR=hello
```

---

## Auto Stop / Auto Start

Fly can automatically:
- **Stop** machines when no traffic for N seconds (saves cost)
- **Start** machines when a new request arrives (~300ms cold start)

Configure in `fly.toml`:

```toml
[http_service]
  auto_stop_machines = true
  auto_start_machines = true
  min_machines_running = 0   # allow 0 running when idle
```

Set `min_machines_running = 1` if you need guaranteed low latency (always-on).

---

## SSH into a Machine

```bash
fly ssh console             # Connect to any running machine
fly ssh console -s          # Select a specific machine interactively
fly ssh console --pty -C "env"  # Run a command and return
```

---

## Machine Metadata & Identity

Inside any Machine, you can access identity via environment variables:

```bash
echo $FLY_APP_NAME       # my-app
echo $FLY_MACHINE_ID     # abc123
echo $FLY_REGION         # ams
echo $FLY_IMAGE_REF      # registry.fly.io/my-app:deployment-xyz
```

Or via the Fly metadata API (available inside Machines only):

```bash
curl http://[fdaa::3]:8080/v1/metadata
```

---

## Tasks

### Task 1 — Inspect a Machine
1. Deploy any app.
2. Run `fly machine list` and note the Machine ID.
3. Run `fly machine status <id>` for full details.

### Task 2 — Stop and Start a Machine
1. Run `fly machine stop <id>`.
2. Confirm `fly status` shows it as stopped.
3. Run `fly machine start <id>` and confirm it recovers.

### Task 3 — SSH into a Machine
1. Run `fly ssh console`.
2. Execute `env | grep FLY` to see all Fly-injected variables.
3. Run `ps aux` to inspect running processes.

---

## Next Steps

→ Continue to [`500`](../500/README.md) — Configuration (`fly.toml`)
