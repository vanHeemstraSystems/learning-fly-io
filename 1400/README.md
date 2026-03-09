# 1400 — Fly Sprites (AI Sandboxes)

## What are Fly Sprites?

[Fly Sprites](https://sprites.dev) are **hardware-isolated sandboxes** designed for running AI-generated or untrusted code safely. Each Sprite is a self-contained environment powered by a Fly Machine.

Key features:
- **Sub-second startup** (~300ms)
- **Hardware isolation** (KVM, not just containers)
- **Checkpointing** — snapshot the VM state, run code, restore if needed
- **Per-second billing** for CPU + memory
- **Private filesystem** per Sprite
- **Built-in networking** with Fly's private mesh

---

## Why Sprites for AI?

Traditional approaches to running AI-generated code (e.g., code interpreters, agent tools) have problems:
- Shared runtimes = noisy neighbors
- Container isolation ≠ hardware isolation (escape risks)
- Cold starts too slow for interactive use

Sprites solve this with VM-level isolation that starts fast enough to be practical.

---

## Core Concepts

### Sprite
A single isolated VM. Each AI action, each user, or each task can get its own Sprite.

### Checkpoint
A snapshot of the entire VM state (memory + disk). You can:
- Checkpoint before running risky code
- Restore if code crashes or corrupts state
- Fork a Sprite from a checkpoint (multiple users from one base image)

### Ephemeral vs. Persistent
- **Ephemeral Sprites**: run and die (stateless execution)
- **Persistent Sprites**: retain state across interactions (agent memory)

---

## Use Cases

| Use case | Description |
|----------|-------------|
| AI code interpreter | Run Python/JS code from LLM in isolation |
| Agent tool execution | Give agents a persistent "computer" |
| Interactive notebooks | Each user gets their own VM |
| Security research | Run unknown binaries safely |
| CI/CD sandboxes | Run untrusted build scripts |

---

## Sprites API (Overview)

Sprites are controlled via the [Sprites API](https://sprites.dev) and/or the Machines API.

### Create a Sprite (Machine)

```bash
curl -X POST \
  "https://api.machines.dev/v1/apps/my-sandbox-app/machines" \
  -H "Authorization: Bearer $(fly auth token)" \
  -H "Content-Type: application/json" \
  -d '{
    "config": {
      "image": "python:3.12-slim",
      "guest": { "cpu_kind": "shared", "cpus": 1, "memory_mb": 512 },
      "init": { "exec": ["/bin/sleep", "3600"] }
    },
    "region": "ams"
  }'
```

### Exec code inside a Sprite

```bash
fly ssh console -a my-sandbox-app -C "python3 -c 'print(2+2)'"
```

Or via the Machines API exec endpoint:

```bash
curl -X POST \
  "https://api.machines.dev/v1/apps/my-sandbox-app/machines/<id>/exec" \
  -H "Authorization: Bearer $(fly auth token)" \
  -H "Content-Type: application/json" \
  -d '{"cmd": ["python3", "-c", "print(42)"], "timeout": 10}'
```

---

## Checkpointing

```bash
# Via CLI (experimental):
fly machine cordon <machine-id>     # Drain traffic
# Snapshot via Machines API
curl -X POST "https://api.machines.dev/v1/apps/<app>/machines/<id>/snapshot" \
  -H "Authorization: Bearer $(fly auth token)"
```

Restore from snapshot:
```bash
fly machine clone <id> --from-snapshot <snapshot-id>
```

---

## Practical Pattern: AI Code Execution Service

```
User sends code to your app
        │
        ▼
Your app creates a Sprite (POST /machines)
        │
        ▼
Your app execs code (POST /machines/<id>/exec)
        │
        ▼
Returns stdout/stderr to user
        │
        ▼
Destroy Sprite (DELETE /machines/<id>)
```

Node.js sketch:
```javascript
async function executeCode(code) {
  // 1. Create a Machine
  const machine = await createMachine('python:3.12-slim');
  
  try {
    // 2. Exec the user's code
    const result = await execInMachine(machine.id, ['python3', '-c', code]);
    return { stdout: result.stdout, stderr: result.stderr };
  } finally {
    // 3. Destroy (cleanup)
    await destroyMachine(machine.id);
  }
}
```

---

## Tasks

### Task 1 — Explore sprites.dev
1. Visit [https://sprites.dev](https://sprites.dev).
2. Try the interactive demo.
3. Note the startup time and isolation model.

### Task 2 — Create an on-demand execution Machine
1. Create a new Fly app: `fly apps create my-sandbox`.
2. Via the Machines API, start a Python Machine.
3. Exec `python3 -c "import sys; print(sys.version)"` inside it.
4. Destroy the Machine.

---

## Next Steps

→ Continue to [`1500`](../1500/README.md) — CI/CD Integration
