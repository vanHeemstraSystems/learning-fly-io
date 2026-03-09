# 1200 — Private Networking (WireGuard / 6PN)

## Fly's Private Network: 6PN

Every Fly Organization has a **private IPv6 network** called **6PN** (Six Private Network). All Machines within the same organization are connected to this network automatically via **WireGuard**.

Key properties:
- All traffic between Machines is **end-to-end encrypted** (WireGuard)
- No public internet involved for intra-org communication
- Uses IPv6 addresses in the `fdaa::/8` range
- Works across regions (Amsterdam → London is private)

---

## DNS on the Private Network

Fly provides an internal DNS resolver at `fdaa::3` that resolves `.internal` hostnames.

### Hostname formats:

```
<app-name>.internal
→ All healthy machines for that app (round-robin)

<region>.<app-name>.internal  
→ All machines in a specific region

<machine-id>.<app-name>.internal
→ A specific machine

top1.nearest.<app-name>.internal
→ The single nearest machine (useful for primary DB routing)
```

### Example: API calling a DB
From inside `my-api` (an app), calling `my-postgres` (another app):

```bash
# From inside my-api machine:
psql "postgresql://postgres:pass@my-postgres.internal:5432/mydb"
```

---

## Checking Private IPs

From inside a Machine:
```bash
# Show all IPs for an app on 6PN
fly dig aaaa my-app.internal

# Show machines in a region
fly dig aaaa ams.my-app.internal
```

Or via environment:
```bash
echo $FLY_PRIVATE_IP   # This Machine's private IPv6
```

---

## WireGuard for Local Access

You can join your **local machine** to the Fly private network:

### Create a WireGuard config
```bash
fly wireguard create personal ams my-dev-laptop
# Saves a .conf file you import into WireGuard
```

### List peers
```bash
fly wireguard list
```

### Remove a peer
```bash
fly wireguard remove my-dev-laptop
```

Once connected via WireGuard, you can reach `<app>.internal` directly from your laptop.

---

## fly proxy (Simpler Alternative)

For quick local access to a private service (without full WireGuard setup):

```bash
fly proxy 5432 -a my-postgres-app
# Tunnels localhost:5432 → my-postgres-app private network port 5432
```

```bash
fly proxy 6379 -a my-redis-app
# Tunnels localhost:6379 → Redis
```

This is the most common way to access Postgres locally:
```bash
fly proxy 5432 -a my-db &
psql -h localhost -p 5432 -U myuser mydb
```

---

## Service-to-Service Communication Pattern

```toml
# In my-worker/fly.toml
[env]
  API_URL = "http://my-api.internal:8080"
```

```javascript
// In my-worker code
const res = await fetch(`${process.env.API_URL}/tasks`);
```

No credentials needed for the network layer — WireGuard handles encryption. Add application-level auth (API keys, JWT) as appropriate.

---

## Tasks

### Task 1 — Confirm 6PN connectivity
1. Deploy two apps in the same org (e.g., `my-api` and `my-worker`).
2. SSH into `my-worker`: `fly ssh console -a my-worker`
3. Run: `curl http://my-api.internal:8080`
4. Confirm the response arrives without public internet.

### Task 2 — Use fly proxy
1. Run: `fly proxy 5432 -a my-postgres-app`
2. In another terminal: `psql -h localhost -p 5432 -U postgres`
3. Run `\l` and exit.

---

## Next Steps

→ Continue to [`1300`](../1300/README.md) — Monitoring, Metrics & Logs
