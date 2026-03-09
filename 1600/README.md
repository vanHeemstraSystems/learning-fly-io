# 1600 — Multi-Region & Distributed Apps

## Why Multi-Region?

- **Latency**: Users in Sydney get responses from Sydney, not Amsterdam
- **Availability**: One region going down doesn't take out your app
- **Compliance**: Keep data in specific jurisdictions

---

## The Global Routing Model

```
User in Tokyo
    │
    ▼ DNS lookup → Fly Anycast IP
Fly Edge (nearest PoP)
    │
    ▼ route to nearest healthy Machine
Machine in nrt (Tokyo)  ← preferred
    or
Machine in sin (Singapore)  ← fallback
```

Fly routes to the **nearest healthy Machine** automatically. No configuration needed for read traffic.

---

## Deploying to Multiple Regions

```bash
fly scale count 1 --region ams     # Amsterdam (primary)
fly scale count 1 --region lhr     # London
fly scale count 1 --region iad     # Washington DC
fly scale count 1 --region syd     # Sydney

fly status   # Confirm 4 machines in 4 regions
```

---

## The Write Problem: Primary Region

For stateful apps with a database, writes must go to the **primary** (where the write-leader is). Reads can go anywhere.

### fly-replay: Cross-Region Write Forwarding

When a Machine in `lhr` receives a write request but only has a read replica:

1. The Machine replies with `fly-replay: region=ams`
2. Fly's proxy intercepts this response
3. Fly replays the entire request to a Machine in `ams` (the primary)
4. The `ams` Machine handles the write and returns the response
5. The user gets the response — transparently

```python
# Python/Django example
from django.db import connection

def handle_request(request):
    if request.method == 'POST':
        if os.getenv('FLY_REGION') != PRIMARY_REGION:
            # We're not in the primary region, replay there
            response = HttpResponse()
            response['fly-replay'] = f'region={PRIMARY_REGION}'
            return response
    # Handle normally
    ...
```

---

## Postgres Multi-Region Setup

### Create replica in another region
```bash
fly machine clone <primary-machine-id> \
  -a my-postgres-app \
  --region lhr
```

The new Machine auto-configures as a read replica (Stolon handles this).

### Read from nearest replica
In your app, use the `DATABASE_REPLICA_URL` (if set) for reads:

```javascript
const readPool = new Pool({ connectionString: process.env.DATABASE_REPLICA_URL });
const writePool = new Pool({ connectionString: process.env.DATABASE_URL });
```

Or use `top1.nearest.my-postgres-app.internal:5432` for reads — always resolves to the nearest Postgres Machine.

---

## LiteFS: Distributed SQLite

LiteFS enables SQLite to be distributed across regions with real-time replication:

```
Primary (ams) ──── write ────► LiteFS primary
Replica (lhr)  ◄── replicate ── LiteFS replica
Replica (syd)  ◄── replicate ── LiteFS replica
```

Reads are fast (local SQLite). Writes that hit a replica are forwarded to the primary via `fly-replay`.

See [`700/README.md`](../700/README.md) for LiteFS setup.

---

## Distributed Architecture Patterns

### Pattern 1: Stateless API (easiest)
All regions are equal. Any machine handles any request.
```
fly scale count 1 --region ams
fly scale count 1 --region lhr
fly scale count 1 --region iad
```
Works perfectly for stateless apps.

### Pattern 2: Primary-replica with replay
Primary region handles writes; replicas handle reads + replay writes.
```
PRIMARY_REGION=ams
→ ams: write + read
→ lhr, iad, syd: read only + replay writes to ams
```

### Pattern 3: Regional sharding
Route different users to different regions based on their data.
Use `fly-force-instance-id` header to pin a user to a specific Machine.

---

## Tasks

### Task 1 — Deploy a stateless app to 3 regions
1. `fly scale count 1 --region ams`
2. `fly scale count 1 --region lhr`
3. `fly scale count 1 --region sin`
4. Run `fly status` and confirm 3 machines.
5. Use a VPN to connect from different locations and measure latency difference.

### Task 2 — Implement fly-replay
1. Modify your app to return `fly-replay: region=<primary>` for POST requests if `FLY_REGION != PRIMARY_REGION`.
2. Deploy to 2 regions.
3. Test with a POST request — confirm it succeeds even when hitting the replica region.

---

## Next Steps

→ Continue to [`1700`](../1700/README.md) — Security & Enterprise Features
