# 1700 — Security & Enterprise Features

## Security Architecture

Fly.io is built on a **memory-safe stack** (Rust + Go) and uses **KVM hardware virtualization** for every Machine — providing true isolation between tenants.

---

## Hardware-Level Isolation

Unlike shared container platforms, Fly Machines run in **KVM virtual machines**:

```
Tenant A Machine      Tenant B Machine
     │                      │
  KVM VM                 KVM VM
     │                      │
  ───┴──────────────────────┴───
         Bare Metal Host
```

KVM provides:
- Separate kernel per Machine
- Hardware memory isolation
- Network namespace isolation
- No container escape to neighbor tenants

---

## Network Security

### All private traffic is WireGuard encrypted
Traffic between Machines on 6PN is encrypted with WireGuard (modern, audited, fast cryptography).

### TLS everywhere public
All public endpoints use TLS. `force_https = true` in `fly.toml` ensures no plain HTTP.

### No inbound ports by default
Machines have no inbound ports open unless you explicitly configure them in `fly.toml` services. A Machine with no `[services]` block is only reachable from the private network.

---

## Secrets Management

- Secrets are encrypted at rest using Fly's key management infrastructure
- Secrets are never exposed in `fly status`, `fly config show`, or logs
- Only the secret **names** are listed; values are never retrievable after setting
- Secrets are injected as environment variables at Machine start time

```bash
fly secrets set API_KEY=secret-value   # Encrypted immediately
fly secrets list                       # Shows names only, never values
```

---

## Access Control (Organizations & Roles)

### Organization roles:
- **Owner** — full control, billing access
- **Admin** — manage apps, members; no billing
- **Member** — deploy and manage apps; no member management
- **Billing Manager** — billing only

### Manage members:
```bash
fly orgs show my-org         # List org details and members
fly orgs invite email@x.com  # Invite a member
```

---

## Deploy Tokens (Scoped Access)

For CI/CD, create scoped tokens instead of using your personal auth token:

```bash
# Deploy-only token (can deploy, cannot delete apps)
fly tokens create deploy -a my-app

# Read-only token
fly tokens create readonly -a my-app

# Org-level token
fly tokens create org -o my-org
```

### Rotate tokens
```bash
fly tokens list
fly tokens revoke <token-id>
```

---

## Single Sign-On (Enterprise)

Enterprise organizations can configure **SSO** with:
- Google Workspace
- Okta
- Azure AD
- Any SAML 2.0 / OIDC provider

Contact Fly.io sales for enterprise SSO setup.

---

## SOC 2 Type 2

Fly.io is **SOC 2 Type 2 attested**. Request the report from Fly.io for compliance purposes.

---

## Compliance Considerations

| Requirement | Fly.io Support |
|-------------|---------------|
| Data residency | Choose specific regions |
| Encryption in transit | TLS + WireGuard (automatic) |
| Encryption at rest | Volumes encrypted at rest |
| Audit logs | Available via Fly dashboard |
| SOC 2 Type 2 | Available on request |
| GDPR | EU regions available (ams, lhr, fra, cdg) |

---

## CRA (Cyber Resilience Act) Considerations

For EU-regulated software deploying on Fly.io:
- Use EU regions (`ams`, `lhr`, `fra`, `cdg`, `waw`)
- Enable TLS everywhere
- Use private networking (6PN) for inter-service communication
- Store secrets in `fly secrets` (not env files)
- Enable Volume encryption and snapshots
- Set up audit logging via log drain

---

## Tasks

### Task 1 — Create a deploy token
1. Run `fly tokens create deploy -a my-app`.
2. Store it in a password manager.
3. Use it in a `fly deploy` command: `FLY_API_TOKEN=<token> fly deploy`.

### Task 2 — Review your org members
1. Run `fly orgs show personal`.
2. Note the members and their roles.

### Task 3 — Audit your secrets
1. Run `fly secrets list`.
2. Identify any secrets that are no longer needed.
3. Remove them with `fly secrets unset <KEY>`.

---

## Next Steps

→ Continue to [`1800`](../1800/README.md) — Pricing & Cost Optimization
