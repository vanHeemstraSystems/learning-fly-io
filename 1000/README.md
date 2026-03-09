# 1000 — Zero-Downtime Deploys

## How Fly Deploys Work

When you run `fly deploy`, Fly performs a **rolling deployment** by default:

```
Old Machine v1 (running)
New Machine v2 (starting)
    │
    ▼ health check passes
New Machine v2 (healthy)
    │
    ▼ Fly proxy shifts traffic
Old Machine v1 (stopped / destroyed)
```

At no point is there a gap in service — the old Machine handles traffic until the new one is proven healthy.

---

## Deployment Strategies

### Rolling (default)
Replaces Machines one at a time. Zero downtime.

```bash
fly deploy
```

### Bluegreen
Creates all new Machines before cutting over. More resource-intensive but instant cutover.

```bash
fly deploy --strategy bluegreen
```

### Canary
Sends a small % of traffic to the new version before full rollout.

```bash
fly deploy --strategy canary
```

### Immediate
Stops old, starts new without health check waiting (fastest but brief downtime).

```bash
fly deploy --strategy immediate
```

---

## Health Checks & Deploy Safety

Fly won't mark a Machine as healthy (and won't cut over traffic) until health checks pass.

```toml
[[http_service.checks]]
  grace_period = "5s"     # Wait before checking after start
  interval = "10s"
  method = "GET"
  path = "/health"
  timeout = "2s"
```

Your app must return `HTTP 200` on `/health` before Fly considers the deploy successful.

---

## Release Commands

Run a command **before** new Machines start (e.g., database migrations):

```toml
[deploy]
  release_command = "python manage.py migrate"
```

The release command runs in a temporary Machine using the new image. If it fails (non-zero exit), the deploy is aborted.

This is the correct pattern for database migrations — run migrations before routing traffic to new app code.

---

## Rollback

### Instant rollback to previous image
```bash
fly deploy --image registry.fly.io/my-app:<previous-deployment-tag>
```

### List previous deployments
```bash
fly releases
```

Output:
```
VERSION  STABLE  TYPE     STATUS   DESCRIPTION           USER     DATE
v10      true    release  complete deploy                 user@ex  1h ago
v9       true    release  complete deploy                 user@ex  3h ago
```

### Rollback to a specific version
```bash
fly deploy --image registry.fly.io/my-app:deployment-01ABCD
```

---

## Wait for Deploy

```bash
fly deploy --wait-timeout 120   # Wait up to 120s for deploy to complete
```

---

## Monitoring a Deploy

```bash
fly logs           # Watch live logs during deploy
fly status         # Check Machine states
```

---

## Deploy from a Specific Docker Image

```bash
fly deploy --image nginx:1.25
fly deploy --image ghcr.io/myorg/myapp:v1.2.3
```

---

## Tasks

### Task 1 — Observe a rolling deploy
1. Deploy your app.
2. Modify the app (change a response message).
3. Run `fly deploy` and in a parallel terminal run `fly logs` — observe the deploy process.
4. In a third terminal, curl your app repeatedly during the deploy: `watch -n 0.5 curl -s https://my-app.fly.dev` — confirm no errors.

### Task 2 — Add a release command
1. Add a simple release command (e.g., `echo "Migration complete"`) to `fly.toml`:
   ```toml
   [deploy]
     release_command = "echo 'Running migrations...'"
   ```
2. Deploy and watch the logs — you should see the release command output.

### Task 3 — Rollback
1. Make a breaking change to your app.
2. Deploy it.
3. Run `fly releases` to find the previous version.
4. Roll back with `fly deploy --image registry.fly.io/<app>:<old-tag>`.

---

## Next Steps

→ Continue to [`1100`](../1100/README.md) — Volumes & Persistent Storage
