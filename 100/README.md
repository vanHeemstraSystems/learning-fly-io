# 100 — Introduction & Core Concepts

## What is Fly.io?

Fly.io is a **developer-first deployment platform** that removes infrastructure complexity while giving you full control over compute. At its heart, it runs your code on **Fly Machines** — hardware-isolated VMs built on KVM — deployed close to your users across a global network.

---

## Core Concepts

### 1. Fly Machines
A Fly Machine is the fundamental unit of compute on Fly.io.

- Built on **KVM hardware virtualization** (not just containers)
- Starts in **~300ms** — fast enough to boot on demand per HTTP request
- Runs **any OCI-compatible container image**
- Billed per second for CPU + memory consumed
- Can be paused when idle and resumed instantly

```
Your App Container
       ↓
  Fly Machine (KVM VM)
       ↓
  Fly.io Metal (bare-metal servers)
       ↓
  Region (e.g., ams, lhr, syd, iad)
```

### 2. Fly Apps
A **Fly App** is a named, logical grouping of Machines and their associated config, networking, and secrets. One app can run many machines.

### 3. Fly Organizations
An **Organization** is the billing and access control boundary. Teams share an org; personal projects live in your personal org.

### 4. Regions
Fly.io operates in **18+ regions** worldwide (e.g., `ams` = Amsterdam, `lhr` = London, `syd` = Sydney, `iad` = Washington DC). You choose which regions your Machines run in.

### 5. `fly.toml`
The configuration file for a Fly App. Lives at the root of your project. Defines build settings, services, ports, health checks, scaling, and more.

### 6. flyctl (the CLI)
The primary way to interact with Fly.io. Install it, log in, and use commands like `fly launch`, `fly deploy`, `fly status`, `fly logs`.

---

## Architecture Overview

```
Developer
   │
   ▼
flyctl / CI pipeline
   │
   ▼ fly deploy
Fly Build System (Dockerfile → OCI image)
   │
   ▼
Fly Registry (private image registry)
   │
   ▼
Fly Machines (KVM VMs, per region)
   │
   ▼
Fly Anycast Network → Users worldwide
```

---

## Key Differentiators vs. Other Platforms

| Feature | Fly.io | Heroku | AWS ECS |
|---------|--------|--------|---------|
| VM startup time | ~300ms | Seconds | Minutes |
| Hardware isolation | KVM | Container | Container |
| Global deployment | Built-in | Add-on | Manual |
| Pricing model | Per-second | Per-dyno | Per-hour |
| Postgres | Managed | Managed | DIY |
| Private networking | Built-in WireGuard | No | VPC |
| Dockerfile support | Native | Buildpacks | Native |

---

## Mental Model

Think of Fly.io as **"your own mini cloud, managed for you"**:

- You bring a Dockerfile (or use their buildpacks)
- They handle the global network, certificates, routing, and hardware
- You control the regions, scaling, and configuration
- Machines are ephemeral by default but can use Volumes for persistence

---

## Next Steps

→ Continue to [`200`](../200/README.md) — Installation & CLI (`flyctl`)
