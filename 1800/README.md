# 1800 — Pricing & Cost Optimization

## Pricing Model

Fly.io charges **per second** for the resources your Machines consume. When a Machine is stopped, you pay only for the Volume storage — not CPU or RAM.

See current pricing at: [fly.io/pricing](https://fly.io/pricing)

---

## What You Pay For

| Resource | Notes |
|----------|-------|
| Machine CPU + RAM | Per second when running |
| Volumes | Per GB per month |
| Outbound bandwidth | Per GB (inbound free) |
| IPv4 addresses | ~$2/month per dedicated IPv4 |
| Postgres Machines | Same as regular Machines |
| Fly Postgres storage | Volume pricing |

---

## Free Allowances (Hobby Tier)

Fly.io provides monthly free allowances:

- **3 shared-cpu-1x 256MB Machines** (always-on)
- **3 GB persistent Volume storage**
- **160 GB outbound data transfer**
- **Shared IPv4 addressing** (free)
- **10 active certificates**

> Check [fly.io/pricing](https://fly.io/pricing) for current free tier limits.

---

## Cost Optimization Strategies

### 1. Auto stop / auto start (most impactful)

```toml
[http_service]
  auto_stop_machines = true
  auto_start_machines = true
  min_machines_running = 0
```

A Machine that sleeps when idle costs **nothing for CPU/RAM** while stopped.

For a hobby app with sporadic traffic: this alone can reduce costs by 90%+.

### 2. Use shared CPUs for low-traffic apps

```toml
[[vm]]
  cpu_kind = "shared"
  cpus = 1
  memory = "256mb"
```

Shared CPUs are ~10x cheaper than dedicated CPUs.

### 3. Right-size memory

Most simple web apps run fine on 256MB. Only increase memory when you see OOM kills:

```bash
fly scale memory 256    # Start small
fly logs | grep -i oom  # Watch for OOM signals
```

### 4. Minimize IPv4 usage

Shared IPv4 is free. Only allocate a dedicated IPv4 if required:
```bash
fly ips list   # Check what you're paying for
fly ips release <ip>   # Release unused dedicated IPs
```

### 5. Delete unused apps and volumes

```bash
fly apps list              # Audit running apps
fly volumes list           # Audit volumes
fly apps destroy old-app   # Clean up unused apps
fly volumes destroy <id>   # Clean up unused volumes
```

### 6. Use the free tier Machines for dev/staging

Keep dev/staging on `shared-cpu-1x / 256MB` with auto-stop. Only use larger VMs in production.

---

## Estimating Costs

### Example: Simple web app

Assumptions:
- 1 Machine, `shared-cpu-1x`, 256MB RAM
- Auto-stop enabled
- Runs 4 hours/day (16 hours idle/stopped)
- 10 GB Volume
- 10 GB outbound transfer

Monthly estimate:
```
Machine (4h/day × 30 days × price/hr)   ~$1-3
Volume (10GB)                            ~$1.50
Outbound (10GB)                          ~$0.30
Total                                    ~$3-5/month
```

### Example: Production API (always-on, 3 regions)

Assumptions:
- 3 Machines, `performance-1x` (2GB RAM), 3 regions
- Always running (min_machines_running = 1)
- 50 GB Volume (Postgres)

Monthly estimate:
```
3x Machines @ ~$15/month each          ~$45
Postgres 50GB Volume                   ~$7.50
Outbound (50GB)                        ~$1.50
Total                                  ~$54/month
```

---

## Monitoring Costs

```bash
fly dashboard       # Opens Fly dashboard → Billing section
```

Set up billing alerts via the dashboard to notify when monthly spend exceeds a threshold.

---

## Tasks

### Task 1 — Audit your current spend
1. Open `fly dashboard` and navigate to Billing.
2. Review the current month's usage breakdown.
3. Identify your top cost driver.

### Task 2 — Enable auto-stop on all dev apps
1. Run `fly apps list`.
2. For each dev/staging app, check `fly.toml` and enable `auto_stop_machines = true`.
3. Redeploy and confirm Machines go to sleep when idle.

### Task 3 — Release unused IPs
1. Run `fly ips list` for each app.
2. Release any dedicated IPv4 you don't need: `fly ips release <ip>`.

---

## Next Steps

→ Continue to [`1900`](../1900/README.md) — Reference & Cheat Sheets
