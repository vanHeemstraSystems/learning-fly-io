# 1100 — Volumes & Persistent Storage

## The Problem with Ephemeral Machines

By default, Fly Machines are **stateless** — when a Machine is replaced (on deploy), all data written to the container's filesystem is lost. For persistent data, you need a **Fly Volume**.

---

## Fly Volumes

A Fly Volume is a **persistent block storage device** attached to a single Machine in a specific region. Think of it as a hard disk for your Machine.

- Built on **NVMe SSDs** (fast, low-latency)
- Tied to one region
- One Volume per Machine (1:1 relationship)
- Survives Machine restarts and deploys
- Does NOT survive `fly volumes destroy`

---

## Creating Volumes

```bash
fly volumes create myapp_data \
  --region ams \
  --size 10          # 10 GB
```

### With replication (across 2 zones)
```bash
fly volumes create myapp_data \
  --region ams \
  --size 10 \
  --count 2          # Creates 2 volumes in different availability zones
```

---

## Mounting in fly.toml

```toml
[[mounts]]
  source = "myapp_data"     # Volume name (not ID)
  destination = "/data"     # Mount path inside container
```

After mounting, your app can read/write `/data` like a normal filesystem.

---

## Volume Management

```bash
fly volumes list                        # List all volumes
fly volumes show <volume-id>            # Show volume details
fly volumes extend <volume-id> --size 20  # Grow to 20GB (can't shrink)
fly volumes destroy <volume-id>         # Delete permanently
```

---

## Snapshots (Backups)

```bash
fly volumes snapshots list <volume-id>      # List snapshots
fly volumes snapshots create <volume-id>    # Manual snapshot
```

Fly takes automatic daily snapshots (retained for 5 days).

### Restore from snapshot
```bash
fly volumes create restored_data \
  --from-snapshot <snapshot-id> \
  --region ams \
  --size 10
```

---

## Multi-Region Volume Strategy

A Volume exists in **one region only**. For a 3-region app:

```bash
fly volumes create myapp_data --region ams --size 10
fly volumes create myapp_data --region lhr --size 10
fly volumes create myapp_data --region syd --size 10
```

The Machine in each region gets its own Volume. Data is NOT replicated between volumes automatically — use LiteFS or Postgres for replicated state.

---

## Common Use Cases

| Use case | Solution |
|----------|---------|
| SQLite database | Volume at `/data/app.db` |
| User uploaded files | Volume at `/data/uploads` |
| Redis data (self-hosted) | Volume at `/data/redis` |
| Logs | Volume at `/data/logs` |
| App config files | Volume at `/data/config` |

---

## Example: SQLite App

`fly.toml`:
```toml
[[mounts]]
  source = "sqlite_data"
  destination = "/data"

[env]
  DATABASE_PATH = "/data/app.db"
```

App code (Node.js):
```javascript
const Database = require('better-sqlite3');
const db = new Database(process.env.DATABASE_PATH);
```

---

## Tasks

### Task 1 — Create and use a Volume
1. `fly volumes create test_data --region ams --size 1 -a my-app`
2. Add `[[mounts]]` to `fly.toml`.
3. Deploy.
4. SSH in: `fly ssh console`
5. Run: `echo "hello persistent world" > /data/test.txt`
6. Exit, redeploy, SSH back in, run: `cat /data/test.txt` — data should persist.

### Task 2 — Snapshot a Volume
1. Run `fly volumes snapshots create <vol-id>`.
2. Run `fly volumes snapshots list <vol-id>` to see it.

---

## Next Steps

→ Continue to [`1200`](../1200/README.md) — Private Networking (WireGuard / 6PN)
