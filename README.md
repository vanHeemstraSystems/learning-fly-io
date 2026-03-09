# learning-fly-io

A structured learning repository for [Fly.io](https://fly.io) — the platform for deploying app servers close to users, running any code fearlessly with hardware-virtualized containers (Fly Machines).

---

## What is Fly.io?

Fly.io is a developer platform built on **Fly Machines** — hardware-virtualized containers (using KVM) that launch in milliseconds and run only when needed. It lets you:

- Deploy any app (Docker container, Rails, Phoenix, Django, Laravel, Node, .NET, and more)
- Scale globally across 18+ regions
- Run isolated sandboxes for AI-generated code (Sprites)
- Use managed Postgres, private networking, and autoscaling — all without Kubernetes complexity

Key principles:
- **Machines as primitives** — VMs that start fast enough to handle HTTP requests
- **Run close to users** — sub-100ms response times globally
- **Pay for what you use** — billed per second for CPU + memory

---

## Repository Structure

Each numbered directory maps to a learning theme, following the `flows → stories → tasks` methodology.

| Directory | Topic |
|-----------|-------|
| [`100`](./100/README.md) | Introduction & Core Concepts |
| [`200`](./200/README.md) | Installation & CLI (`flyctl`) |
| [`300`](./300/README.md) | Your First Fly App |
| [`400`](./400/README.md) | Fly Machines Deep Dive |
| [`500`](./500/README.md) | Configuration (`fly.toml`) |
| [`600`](./600/README.md) | Networking, Routing & Certificates |
| [`700`](./700/README.md) | Managed Postgres & Storage |
| [`800`](./800/README.md) | Secrets & Environment Variables |
| [`900`](./900/README.md) | Autoscaling & Regions |
| [`1000`](./1000/README.md) | Zero-Downtime Deploys |
| [`1100`](./1100/README.md) | Volumes & Persistent Storage |
| [`1200`](./1200/README.md) | Private Networking (WireGuard / 6PN) |
| [`1300`](./1300/README.md) | Monitoring, Metrics & Logs |
| [`1400`](./1400/README.md) | Fly Sprites (AI Sandboxes) |
| [`1500`](./1500/README.md) | CI/CD Integration |
| [`1600`](./1600/README.md) | Multi-Region & Distributed Apps |
| [`1700`](./1700/README.md) | Security & Enterprise Features |
| [`1800`](./1800/README.md) | Pricing & Cost Optimization |
| [`1900`](./1900/README.md) | Reference & Cheat Sheets |

---

## How to Use This Repository

1. Start with `100` — Introduction & Core Concepts.
2. Follow directories in order or jump to a topic of interest.
3. Each directory contains:
   - `README.md` — the learning narrative (story)
   - `tasks/` — hands-on exercises
   - `flows/` — workflow diagrams or step sequences (where applicable)
4. Code examples are self-contained and deployable.

---

## Prerequisites

- A [Fly.io account](https://fly.io/app/sign-up) (free tier available)
- Docker installed locally (optional but recommended)
- Basic CLI familiarity

---

## References

- [Fly.io Docs](https://fly.io/docs/)
- [Fly.io Community](https://community.fly.io/)
- [Fly.io GitHub](https://github.com/superfly/)
- [Fly.io Status](https://status.flyio.net/)
- [Fly.io Pricing](https://fly.io/pricing/)
