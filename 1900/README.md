# 1900 — Reference & Cheat Sheets

## flyctl Command Cheat Sheet

### Authentication
```bash
fly auth login                        # Log in
fly auth logout                       # Log out
fly auth whoami                       # Check identity
fly auth token                        # Print current token
```

### App Management
```bash
fly apps list                         # List all apps
fly apps create <name>                # Create app (no deploy)
fly apps destroy <name>               # Destroy app permanently
fly status                            # App status + machines
fly open                              # Open in browser
fly releases                          # List deploy history
```

### Deploying
```bash
fly launch                            # Interactive: create + first deploy
fly deploy                            # Deploy current directory
fly deploy --remote-only              # Build on Fly (no local Docker)
fly deploy --image <image>            # Deploy specific image
fly deploy --strategy bluegreen       # Bluegreen deploy
fly deploy --wait-timeout 120         # Wait up to 120s
```

### Machines
```bash
fly machine list                      # List machines
fly machine status <id>               # Machine details
fly machine start <id>                # Start stopped machine
fly machine stop <id>                 # Stop machine
fly machine restart <id>              # Restart machine
fly machine destroy <id>              # Delete machine
fly machine clone <id>                # Clone a machine
fly machine run <image>               # Run a one-off machine
```

### Logs
```bash
fly logs                              # Live logs
fly logs -r ams                       # Filter by region
fly logs --instance <id>              # Specific machine
```

### SSH
```bash
fly ssh console                       # SSH into a machine
fly ssh console -C "command"          # Run command
fly ssh console -s                    # Select machine interactively
```

### Secrets
```bash
fly secrets set KEY=val               # Set a secret
fly secrets set K1=v1 K2=v2           # Set multiple
fly secrets unset KEY                 # Remove a secret
fly secrets list                      # List secret names
fly secrets import < .env             # Import from file
```

### Scaling
```bash
fly scale count 3                     # 3 machines globally
fly scale count 2 --region lhr        # 2 in London
fly scale vm shared-cpu-1x            # Change VM size
fly scale memory 512                  # Set memory (MB)
fly scale show                        # Show current scale
```

### Volumes
```bash
fly volumes create <name> --region ams --size 10
fly volumes list                      # List volumes
fly volumes extend <id> --size 20     # Grow volume
fly volumes destroy <id>              # Delete volume
fly volumes snapshots list <id>       # List snapshots
fly volumes snapshots create <id>     # Manual snapshot
```

### Postgres
```bash
fly postgres create                   # Create DB cluster
fly postgres attach <db-app>          # Attach to current app
fly postgres connect -a <db-app>      # Open psql
fly postgres config -a <db-app>       # Show connection string
```

### Networking
```bash
fly ips list                          # List IPs
fly ips allocate-v4                   # Add dedicated IPv4
fly ips allocate-v6                   # Add IPv6
fly ips release <ip>                  # Release IP
fly certs list                        # List certs
fly certs add <domain>                # Add custom domain cert
fly proxy <port> -a <app>             # Tunnel to private app
```

### WireGuard
```bash
fly wireguard create                  # Create WireGuard peer
fly wireguard list                    # List peers
fly wireguard remove <peer>           # Remove peer
```

### Tokens
```bash
fly tokens create deploy -a <app>     # Deploy token
fly tokens create readonly -a <app>   # Read-only token
fly tokens list                       # List tokens
fly tokens revoke <id>                # Revoke token
```

### Info
```bash
fly platform regions                  # List regions
fly platform vm-sizes                 # List VM sizes
fly config show                       # Show app config
fly config validate                   # Validate fly.toml
fly checks list                       # Health check status
fly dashboard                         # Open web dashboard
```

---

## fly.toml Quick Reference

```toml
app = "my-app"
primary_region = "ams"

[build]
  [build.args]
    NODE_ENV = "production"

[env]
  LOG_LEVEL = "info"

[http_service]
  internal_port = 8080
  force_https = true
  auto_stop_machines = true
  auto_start_machines = true
  min_machines_running = 0
  [http_service.concurrency]
    type = "requests"
    soft_limit = 200
    hard_limit = 250

[[http_service.checks]]
  grace_period = "5s"
  interval = "10s"
  method = "GET"
  path = "/health"
  timeout = "2s"

[[mounts]]
  source = "myapp_data"
  destination = "/data"

[[vm]]
  memory = "512mb"
  cpu_kind = "shared"
  cpus = 1

[deploy]
  release_command = "python manage.py migrate"

[[metrics]]
  port = 9091
  path = "/metrics"

[processes]
  app = "node server.js"
  worker = "node worker.js"
```

---

## Region Codes

| Code | City | Continent |
|------|------|-----------|
| `ams` | Amsterdam | EU |
| `lhr` | London | EU |
| `fra` | Frankfurt | EU |
| `cdg` | Paris | EU |
| `waw` | Warsaw | EU |
| `iad` | Washington DC | US |
| `ord` | Chicago | US |
| `dfw` | Dallas | US |
| `den` | Denver | US |
| `lax` | Los Angeles | US |
| `sjc` | San Jose | US |
| `sea` | Seattle | US |
| `syd` | Sydney | AU |
| `sin` | Singapore | APAC |
| `nrt` | Tokyo | APAC |
| `hkg` | Hong Kong | APAC |
| `gru` | São Paulo | SA |
| `bog` | Bogotá | SA |

---

## Fly-Injected Environment Variables

| Variable | Example Value |
|----------|--------------|
| `FLY_APP_NAME` | `my-app` |
| `FLY_REGION` | `ams` |
| `FLY_MACHINE_ID` | `abc123def456` |
| `FLY_IMAGE_REF` | `registry.fly.io/my-app:deployment-01` |
| `FLY_PUBLIC_IP` | `1.2.3.4` |
| `FLY_PRIVATE_IP` | `fdaa::1` |

---

## Common fly-replay Pattern

```python
PRIMARY_REGION = "ams"

def handle_write(request):
    if os.getenv("FLY_REGION") != PRIMARY_REGION:
        response = make_response("", 307)
        response.headers["fly-replay"] = f"region={PRIMARY_REGION}"
        return response
    # do the actual write
```

---

## Useful Links

| Resource | URL |
|----------|-----|
| Fly.io Docs | https://fly.io/docs/ |
| Fly.io Pricing | https://fly.io/pricing/ |
| Community Forum | https://community.fly.io/ |
| Status Page | https://status.flyio.net/ |
| GitHub (superfly) | https://github.com/superfly/ |
| Sprites | https://sprites.dev/ |
| Managed Postgres | https://fly.io/mpg |

---

*End of learning-fly-io repository.*
