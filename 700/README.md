# 700 — Managed Postgres & Storage

## Fly Postgres Overview

Fly Postgres is a **self-managed Postgres cluster** running on Fly Machines — not a fully managed service like RDS, but one where Fly handles VM provisioning and you maintain the database config. It uses [Stolon](https://github.com/sorintlab/stolon) for high-availability clustering.

Key characteristics:
- Runs as a Fly App in your organization
- Accessible over private 6PN networking
- Supports replication (primary + replicas)
- Supports Postgres extensions
- Snapshots/backups available

---

## Creating a Postgres Cluster

```bash
fly postgres create
```

The wizard asks:
- App name (e.g., `my-app-db`)
- Region
- VM size and count (single node vs HA)
- Volume size (disk)

### Common options:
```bash
fly postgres create \
  --name my-app-db \
  --region ams \
  --initial-cluster-size 1 \
  --vm-size shared-cpu-1x \
  --volume-size 10
```

---

## Attaching Postgres to Your App

```bash
fly postgres attach my-app-db --app my-app
```

This:
1. Creates a database user for your app
2. Sets a `DATABASE_URL` secret in your app automatically

Your app code can now do:
```javascript
const { Pool } = require('pg');
const pool = new Pool({ connectionString: process.env.DATABASE_URL });
```

---

## Connecting Locally (via proxy)

```bash
fly proxy 5432 -a my-app-db
```

Then from your terminal:
```bash
psql -h localhost -p 5432 -U postgres
# or
psql "$(fly postgres config -a my-app-db)"
```

---

## Running SQL

```bash
fly postgres connect -a my-app-db       # Open psql shell
fly postgres connect -a my-app-db -d mydb  # Connect to specific DB
```

Run a query:
```bash
fly postgres connect -a my-app-db -d mydb -c "SELECT count(*) FROM users;"
```

---

## Scaling Postgres

### Upgrade VM size
```bash
fly scale vm performance-1x -a my-app-db
```

### Add a read replica
```bash
fly machine clone <primary-machine-id> -a my-app-db --region lhr
```

### Increase disk
```bash
fly volumes extend <volume-id> --size 20
```

---

## Backups & Snapshots

```bash
fly volumes list -a my-app-db          # List volumes
fly volumes snapshots list <volume-id> # List snapshots
fly volumes snapshots create <volume-id> # Manual snapshot
```

Fly takes daily automatic snapshots.

---

## Fly Volumes (General Storage)

For non-Postgres persistent storage (e.g., uploaded files, SQLite, cache):

### Create a Volume
```bash
fly volumes create myapp_data \
  --region ams \
  --size 10 \
  --app my-app
```

### Mount in fly.toml
```toml
[[mounts]]
  source = "myapp_data"
  destination = "/data"
```

### Important Volume Constraints
- A Volume is **tied to one region** and **one Machine** at a time
- For multi-region apps, create a Volume per region
- If you have 2 Machines in the same region, you need 2 Volumes

---

## SQLite + LiteFS (Distributed SQLite)

For apps using SQLite, Fly offers **LiteFS** — a distributed file system that replicates SQLite across regions in real time.

- Primary node handles writes
- Replicas handle reads
- Automatic failover

Basic LiteFS setup in `fly.toml`:
```toml
[processes]
  app = "litefs mount -- /app/start.sh"

[[mounts]]
  source = "litefs_data"
  destination = "/var/lib/litefs"
```

See [LiteFS docs](https://fly.io/docs/litefs/) for full setup.

---

## Tasks

### Task 1 — Create and attach a Postgres database
1. Run `fly postgres create` (single node, smallest size).
2. Run `fly postgres attach <db-app> --app <your-app>`.
3. Verify `DATABASE_URL` is set: `fly secrets list`.

### Task 2 — Connect locally
1. Run `fly proxy 5432 -a <db-app>`.
2. Open another terminal, run `psql -h localhost -p 5432 -U postgres`.
3. Run `\l` to list databases.

### Task 3 — Create a Volume and mount it
1. Create a volume: `fly volumes create test_data --region ams --size 1`.
2. Add `[[mounts]]` to `fly.toml`.
3. Deploy and SSH in, then run `df -h /data` to confirm the mount.

---

## Next Steps

→ Continue to [`800`](../800/README.md) — Secrets & Environment Variables
