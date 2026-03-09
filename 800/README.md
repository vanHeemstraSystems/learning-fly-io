# 800 — Secrets & Environment Variables

## Two Ways to Pass Config

| Method | Use for | Stored in |
|--------|---------|-----------|
| `[env]` in `fly.toml` | Non-sensitive config | Version control (fly.toml) |
| `fly secrets` | Sensitive values | Fly's encrypted store |

**Never put passwords, API keys, or database URLs in `fly.toml`.**

---

## Managing Secrets

### Set a secret
```bash
fly secrets set MY_API_KEY=abc123
fly secrets set DB_PASSWORD=supersecret SMTP_PASSWORD=another
```

### List secrets (names only — values are never shown)
```bash
fly secrets list
```

Output:
```
NAME          DIGEST    CREATED AT
MY_API_KEY    abc1de2f  1 minute ago
DATABASE_URL  def23abc  2 days ago
```

### Unset a secret
```bash
fly secrets unset MY_API_KEY
```

### Import secrets from a `.env` file
```bash
fly secrets import < .env
```

---

## How Secrets Work

1. You set a secret via CLI.
2. Fly encrypts and stores it.
3. On deploy, secrets are **injected as environment variables** into each Machine.
4. When you update a secret, Fly triggers a rolling restart of all Machines automatically.

```
fly secrets set KEY=value
      │
      ▼
Fly encrypted key store
      │
      ▼ (on deploy or secret update)
Machine environment variables (process env)
      │
      ▼
process.env.KEY (Node.js) / os.environ['KEY'] (Python)
```

---

## Non-Sensitive Environment Variables

Use the `[env]` block in `fly.toml`:

```toml
[env]
  LOG_LEVEL = "info"
  APP_ENV = "production"
  PORT = "8080"
  ALLOWED_ORIGINS = "https://myapp.example.com"
```

These are visible in `fly.toml` (committed to Git), so only use for non-sensitive config.

---

## Accessing in Code

### Node.js
```javascript
const apiKey = process.env.MY_API_KEY;
const dbUrl = process.env.DATABASE_URL;
const logLevel = process.env.LOG_LEVEL || 'info';
```

### Python
```python
import os
api_key = os.environ.get('MY_API_KEY')
db_url = os.environ['DATABASE_URL']
```

### Shell
```bash
echo $MY_API_KEY
```

---

## Fly-Injected Variables

Fly automatically injects these (no need to set them):

| Variable | Value example |
|----------|--------------|
| `FLY_APP_NAME` | `my-fly-app` |
| `FLY_REGION` | `ams` |
| `FLY_MACHINE_ID` | `abc123def456` |
| `FLY_IMAGE_REF` | `registry.fly.io/my-fly-app:deployment-01` |
| `FLY_PUBLIC_IP` | Public IP of the Machine |

---

## Secret Rotation Pattern

When rotating a secret (e.g., API key), use the `--stage` flag to queue without immediately restarting:

```bash
fly secrets set NEW_KEY=value --stage
fly secrets set OLD_KEY=value --stage
# Then trigger deploy manually:
fly deploy
```

---

## .env Files (Local Development)

For local development, use a `.env` file (add to `.gitignore`):

```bash
# .env
DATABASE_URL=postgresql://localhost/myapp_dev
LOG_LEVEL=debug
MY_API_KEY=dev_key_123
```

Use `dotenv` in Node.js or `python-dotenv` in Python to load these locally.

**Never commit `.env` to Git.** Always add it to `.gitignore`.

---

## Tasks

### Task 1 — Set and verify a secret
1. Run `fly secrets set GREETING="Hello from Fly secrets"`.
2. SSH into your app: `fly ssh console`.
3. Run `echo $GREETING` — confirm the value appears.

### Task 2 — Use a secret in your app
1. Modify your app to read `process.env.GREETING` (or `os.environ['GREETING']`).
2. Return it in the HTTP response.
3. Deploy and verify via `fly open`.

### Task 3 — Import from .env
1. Create a `.env` file with 2-3 test variables.
2. Run `fly secrets import < .env`.
3. Run `fly secrets list` to confirm they were imported.

---

## Next Steps

→ Continue to [`900`](../900/README.md) — Autoscaling & Regions
