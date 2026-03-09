# 200 — Installation & CLI (`flyctl`)

## Installing flyctl

### macOS
```bash
brew install flyctl
```

### Linux
```bash
curl -L https://fly.io/install.sh | sh
```

### Windows (PowerShell)
```powershell
pwsh -Command "iwr https://fly.io/install.ps1 -useb | iex"
```

### Verify installation
```bash
fly version
# fly v0.x.x linux/amd64 ...
```

---

## Authentication

```bash
# Log in (opens browser)
fly auth login

# Log out
fly auth logout

# Check who you're logged in as
fly auth whoami
```

---

## Essential flyctl Commands

### App Management
```bash
fly apps list                    # List all your apps
fly apps create my-app-name      # Create a new app (without deploying)
fly apps destroy my-app-name     # Destroy an app permanently
fly open                         # Open app in browser
fly status                       # Show app status & machines
```

### Deploy
```bash
fly launch                       # Interactive: create app + first deploy
fly deploy                       # Deploy current directory
fly deploy --image my/image:tag  # Deploy a specific Docker image
fly deploy --remote-only         # Build remotely on Fly's infrastructure
fly deploy --local-only          # Build locally, push to Fly registry
```

### Machines
```bash
fly machine list                 # List all machines in app
fly machine status <id>          # Status of a specific machine
fly machine start <id>           # Start a stopped machine
fly machine stop <id>            # Stop a running machine
fly machine restart <id>         # Restart a machine
fly machine destroy <id>         # Permanently delete a machine
```

### Logs
```bash
fly logs                         # Live-tail logs
fly logs --instance <id>         # Logs from a specific machine
```

### SSH & Exec
```bash
fly ssh console                  # Open SSH session in a running machine
fly ssh console -C "ls /app"     # Run a command non-interactively
```

### Secrets
```bash
fly secrets set KEY=value        # Set a secret
fly secrets list                 # List secret names (values hidden)
fly secrets unset KEY            # Remove a secret
```

### Scaling
```bash
fly scale count 3                # Scale to 3 machines
fly scale count 1 --region lhr   # 1 machine in London
fly scale vm shared-cpu-1x       # Change VM size
fly scale memory 512             # Set memory to 512MB
```

### Postgres
```bash
fly postgres create              # Create a managed Postgres cluster
fly postgres attach <db-app>     # Attach DB to current app (sets DATABASE_URL)
fly postgres connect -a <db-app> # Open psql session
```

### Networking
```bash
fly ips list                     # List IPs assigned to app
fly ips allocate-v4              # Allocate a dedicated IPv4
fly ips allocate-v6              # Allocate an IPv6 address
fly certs list                   # List TLS certificates
fly certs add my-domain.com      # Add a custom domain certificate
```

### Config & Info
```bash
fly config show                  # Show current app config
fly config save                  # Save remote config to fly.toml
fly regions list                 # Show available regions
fly platform vm-sizes            # List available VM sizes
fly platform regions             # List all regions
```

---

## Helpful Flags (Global)

| Flag | Meaning |
|------|---------|
| `-a <app>` or `--app <app>` | Target a specific app |
| `-r <region>` or `--region <region>` | Target a specific region |
| `--json` | Output in JSON format |
| `--verbose` | More detailed output |

---

## Tasks

### Task 1 — Install and authenticate
1. Install `flyctl` using the appropriate method for your OS.
2. Run `fly auth login`.
3. Run `fly auth whoami` to confirm.

### Task 2 — Explore the CLI
1. Run `fly platform regions` and note the region code closest to you.
2. Run `fly platform vm-sizes` and identify the smallest VM available.
3. Run `fly apps list` to see your existing apps (if any).

---

## Next Steps

→ Continue to [`300`](../300/README.md) — Your First Fly App
