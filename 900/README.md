# 900 — Autoscaling & Regions

## Fly's Autoscaling Philosophy

Fly.io autoscaling works at the **Machine level** — Machines are started and stopped based on demand. Unlike traditional autoscalers that take minutes, Fly Machines can start in ~300ms, making fine-grained autoscaling practical.

Two mechanisms:
1. **Auto start/stop** — automatic based on traffic
2. **Manual scaling** — you control Machine count per region

---

## Auto Start / Stop

Configured in `fly.toml`:

```toml
[http_service]
  auto_stop_machines = true    # Stop when idle
  auto_start_machines = true   # Start on incoming request
  min_machines_running = 0     # Allow full scale-to-zero
```

### How it works:
- When traffic drops to zero, idle Machines stop (no CPU cost)
- When a new request arrives, Fly starts a Machine (~300ms)
- The request is held briefly while the Machine boots

### Always-on (no cold start):
```toml
[http_service]
  auto_stop_machines = false
  min_machines_running = 1
```

---

## Concurrency-Based Scaling

Fly can automatically start new Machines when load per Machine exceeds a threshold:

```toml
[http_service.concurrency]
  type = "requests"      # or "connections"
  soft_limit = 25        # Start new instances at this concurrency
  hard_limit = 30        # Reject above this
```

- `soft_limit` — Fly starts another Machine when concurrency exceeds this
- `hard_limit` — Fly rejects connections above this (returns 503)
- New Machines are started in the same region as the load

---

## Manual Scaling

### Scale Machine count
```bash
fly scale count 3            # 3 Machines globally
fly scale count 2 --region ams    # 2 in Amsterdam
fly scale count 1 --region lhr    # 1 in London
```

### Scale VM size
```bash
fly scale vm shared-cpu-1x   # 1 shared CPU
fly scale vm performance-2x  # 2 dedicated CPUs
fly scale memory 1024         # 1GB RAM
```

### Check current scale
```bash
fly status
fly scale show
```

---

## Multi-Region Deployment

Fly.io has 18+ regions. Deploy to multiple regions for:
- Lower latency for global users
- Redundancy / high availability

### Available regions
```bash
fly platform regions
```

Common regions:

| Code | City |
|------|------|
| `ams` | Amsterdam |
| `lhr` | London |
| `iad` | Washington DC |
| `syd` | Sydney |
| `sin` | Singapore |
| `nrt` | Tokyo |
| `gru` | São Paulo |

### Deploy to a new region
```bash
fly scale count 1 --region lhr
fly scale count 1 --region syd
```

### Remove from a region
```bash
fly scale count 0 --region syd
```

---

## Primary Region

The `primary_region` in `fly.toml` is where:
- New Machines are created by default
- Postgres writes are routed (if using `fly-replay`)
- The app is "home"

```toml
primary_region = "ams"
```

---

## Routing to Nearest Region

Fly automatically routes incoming requests to the **nearest healthy Machine** using Anycast routing. No configuration needed — it just works.

If a region has no running Machine and auto-start is enabled, the request wakes a Machine in that region (or falls back to another region).

---

## Region-Aware Application Design

For stateless apps: multi-region is plug-and-play.

For stateful apps (databases), use the **fly-replay** pattern:
```
Request arrives in lhr
    ↓
Machine in lhr handles read (from local replica)
    ↓
Write request? → Reply with: fly-replay: region=ams
    ↓
Fly proxy retries the request in ams (primary)
```

---

## Tasks

### Task 1 — Enable auto stop/start
1. Set `auto_stop_machines = true` and `min_machines_running = 0` in `fly.toml`.
2. Deploy.
3. Wait ~2 minutes with no traffic, then check `fly status` — Machine should be stopped.
4. Visit your app URL — Machine should wake up.

### Task 2 — Scale to 2 Machines
1. Run `fly scale count 2`.
2. Run `fly status` to confirm 2 Machines are running.
3. Hit your app multiple times and check logs — you should see requests on different Machine IDs.

### Task 3 — Deploy to a second region
1. Run `fly scale count 1 --region lhr` (or another region).
2. Run `fly status` — confirm Machines in 2 regions.
3. Run `fly scale count 0 --region lhr` to remove it.

---

## Next Steps

→ Continue to [`1000`](../1000/README.md) — Zero-Downtime Deploys
