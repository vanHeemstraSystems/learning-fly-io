# 300 — Your First Fly App

## Goal

Deploy a simple Node.js "Hello World" HTTP server to Fly.io from scratch.

---

## Step 1 — Create a Simple App

Create a project directory:

```bash
mkdir my-fly-app && cd my-fly-app
```

Create `app.js`:

```javascript
const http = require('http');

const PORT = process.env.PORT || 8080;

const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end(`Hello from Fly.io! Region: ${process.env.FLY_REGION || 'unknown'}\n`);
});

server.listen(PORT, '0.0.0.0', () => {
  console.log(`Server running on port ${PORT}`);
});
```

Create `package.json`:

```json
{
  "name": "my-fly-app",
  "version": "1.0.0",
  "scripts": {
    "start": "node app.js"
  }
}
```

---

## Step 2 — Create a Dockerfile

```dockerfile
FROM node:20-alpine

WORKDIR /app
COPY package.json .
COPY app.js .

EXPOSE 8080

CMD ["node", "app.js"]
```

---

## Step 3 — Launch on Fly.io

```bash
fly launch
```

The interactive wizard will:
1. Detect your Dockerfile automatically
2. Ask for an app name (or generate one)
3. Ask which organization to use
4. Ask which region to deploy in
5. Ask if you want to set up Postgres (say No for now)
6. Create `fly.toml` and do a first deploy

Sample `fly.toml` that gets generated:

```toml
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

## Step 4 — Check Status

```bash
fly status
```

Expected output:
```
App
  Name     = my-fly-app
  Owner    = personal
  Hostname = my-fly-app.fly.dev
  Image    = my-fly-app:deployment-01...

Machines
ID            PROCESS  VERSION  REGION  STATE    CHECKS  LAST UPDATED
abc123def456  app      1        ams     started  1 total  30s ago
```

---

## Step 5 — Open in Browser

```bash
fly open
# Opens https://my-fly-app.fly.dev in your browser
```

You should see:
```
Hello from Fly.io! Region: ams
```

---

## Step 6 — View Logs

```bash
fly logs
```

---

## Step 7 — Redeploy After a Change

Edit `app.js` to return a different message, then:

```bash
fly deploy
```

Fly builds a new image and performs a zero-downtime rolling deploy.

---

## Step 8 — Destroy the App (Cleanup)

```bash
fly apps destroy my-fly-app
```

---

## Key Environment Variables Fly Injects

| Variable | Description |
|----------|-------------|
| `FLY_APP_NAME` | Your app name |
| `FLY_REGION` | Region code where this Machine is running |
| `FLY_MACHINE_ID` | Unique Machine ID |
| `FLY_IMAGE_REF` | The Docker image reference |
| `PORT` | Port your app should listen on (usually 8080) |

---

## What Just Happened?

```
fly launch
   │
   ├─ Detected Dockerfile
   ├─ Created app in Fly.io registry
   ├─ Built image remotely
   ├─ Started a Fly Machine (KVM VM)
   ├─ Assigned fly.dev subdomain
   └─ Issued TLS certificate (automatic!)
```

---

## Tasks

### Task 1 — Deploy Hello World
Follow all steps above. Confirm `fly open` shows your message.

### Task 2 — Change the message
Modify `app.js`, run `fly deploy`, and verify the updated message in the browser.

### Task 3 — Inspect your Machine
Run `fly machine list` and note the machine ID, region, and state.

---

## Next Steps

→ Continue to [`400`](../400/README.md) — Fly Machines Deep Dive
