# 600 — Networking, Routing & Certificates

## Fly.io Network Model

Every Fly App gets:
- A **`.fly.dev` subdomain** automatically (`my-app.fly.dev`)
- An **IPv6 address** (shared, free)
- **TLS certificate** (issued automatically via Let's Encrypt)
- Access to **Fly's private 6PN network** (inter-app communication)

---

## Public IPs

```bash
fly ips list             # View assigned IPs
fly ips allocate-v4      # Add a dedicated IPv4 (~$2/month)
fly ips allocate-v6      # Add a shared IPv6 (free)
fly ips release <ip>     # Remove an IP
```

> **Tip:** IPv6 is free and is what `.fly.dev` subdomains use. Allocate IPv4 only if you need it for clients that can't use IPv6.

---

## Custom Domains

### Step 1 — Allocate an IP
```bash
fly ips allocate-v4   # Required for custom domain TLS
```

### Step 2 — Point DNS at Fly
In your DNS provider (e.g., Cloudflare, Namecheap), add:
```
Type  Name              Value
A     myapp.example.com <your IPv4>
AAAA  myapp.example.com <your IPv6>
```

### Step 3 — Add the Certificate
```bash
fly certs add myapp.example.com
```

### Step 4 — Verify
```bash
fly certs show myapp.example.com
```

Fly automatically handles certificate renewal.

---

## TLS / HTTPS

`force_https = true` in `fly.toml` ensures all HTTP traffic is redirected to HTTPS.

Fly terminates TLS at the edge (at Fly's Anycast network), forwarding plain HTTP to your Machine's `internal_port`.

```
User ──HTTPS──► Fly Edge (TLS termination) ──HTTP──► Machine :8080
```

---

## Ports and Handlers

In `fly.toml` services, each port has **handlers** that specify what the edge does:

```toml
[[services.ports]]
  port = 443
  handlers = ["tls", "http"]   # Terminate TLS, then proxy HTTP

[[services.ports]]
  port = 80
  handlers = ["http"]          # Plain HTTP (usually redirect to 443)
```

For non-HTTP TCP (e.g., Postgres):
```toml
[[services.ports]]
  port = 5432
  handlers = []                # Pass through raw TCP
```

---

## Private Networking (6PN)

All Fly Machines within an **organization** can communicate with each other over a private IPv6 network called **6PN** (6PN = Fly's WireGuard-based private network).

### DNS Format
```
<app-name>.internal              → resolves to all Machines in the app
<region>.<app-name>.internal     → resolves to Machines in that region
<machine-id>.<app-name>.internal → resolves to a specific Machine
```

### Example: Service-to-Service
If `my-api` and `my-worker` are both apps in the same org:

```javascript
// Inside my-worker, calling my-api privately:
const res = await fetch("http://my-api.internal:8080/api/jobs");
```

No public internet involved — traffic stays on Fly's private WireGuard mesh.

---

## WireGuard VPN (Developer Access)

You can connect your local machine to the Fly private network:

```bash
fly wireguard create           # Create a WireGuard config
fly wireguard list             # List WireGuard peers
fly proxy 5432 -a my-db-app    # Proxy a port locally (e.g., for Postgres)
```

The `fly proxy` command is the most common way to connect to Postgres locally:

```bash
fly proxy 5432 -a my-postgres-app
# Now psql -h localhost -p 5432 -U postgres works!
```

---

## Granular Routing

Fly supports **path-based routing** via `fly.toml` statics and service configuration, and **region-aware routing** via the `fly-prefer-region` header.

### Force a specific region
HTTP clients can send:
```
fly-prefer-region: lhr
```
And Fly will route to a Machine in London if available.

### Replay (cross-region forwarding)
Machines can respond with:
```
fly-replay: region=lhr
```
To instruct the Fly proxy to forward the request to a Machine in another region (useful for primary-region write routing in distributed apps).

---

## Tasks

### Task 1 — Check your app's IPs
Run `fly ips list` and note whether you have IPv4 or IPv6.

### Task 2 — Add a custom domain (optional)
If you own a domain:
1. Allocate an IPv4.
2. Set up DNS.
3. Run `fly certs add yourdomain.com`.
4. Wait for certificate issuance and verify with `fly certs show`.

### Task 3 — Test 6PN (two-app setup)
1. Deploy two apps in the same org.
2. SSH into app A (`fly ssh console`).
3. Run `curl http://<app-b-name>.internal:8080` from inside app A.
4. Confirm the request succeeds without any public internet traffic.

---

## Next Steps

→ Continue to [`700`](../700/README.md) — Managed Postgres & Storage
