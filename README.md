# SecretDGX — spec skeleton (Level 1)

> The **bare-metal, agent-oriented** form. This repo holds the *idea + the
> invariant contracts + the adapter seams* — **no working implementation**.
> A build agent (Codex, Claude, cursor, an LLM in CI, you) reads this and
> rebuilds the app for ITS OWN environment: its own Sparks/workstations/Macs,
> its own IPs, its own network.

If you want a runnable app, use the sibling repos:
- **`SecretDGX-reference`** — a working reference engine against these contracts.
- **`SecretDGX`** — a config-first production instance.

This repo is the **specification** they implement against.

---

## What the product is (the brief)

Turn **every NVIDIA device in a fleet** (DGX Spark GB10, RTX workstations,
Macs with `nvidia-smi`) into a **live macOS dock icon**:

- one icon per node, redrawn every ~2s
- shows node name + 3 thin bars: **GPU % / CPU % / MEM %** (green/yellow/red)
- a **throttle dot** when the GPU is clock-throttled
- offline nodes → greyed **✕**
- **click** → a glass detail panel: full readout + ssh alias

It is a *status surface*, not a job scheduler and not a dashboard-in-a-browser.

## The invariant contracts

Read `CONTRACTS.md` first. The non-negotiables (do not renegotiate in an implementation):

1. **One config file drives everything** — the node set, described by `contracts/node.schema.json`.
2. **Node identity is per-bundle / per-launch, never per-build** — a single shared
   binary serves any node; identity comes from bundle-id / launch token.
3. **SSH is the only transport**, one short call per node per refresh.
4. **Lifecycle is repeatable + reversible**: discover → authorize → build → run → uninstall.
5. **GB10 unified memory, not VRAM** — `nvidia-smi` vram reads N/A on GB10; read `/proc/meminfo`.

## The adapter seams (what the build agent implements per-environment)

`adapters/` defines the three seams an agent must fill. Everything else follows
from the contracts. The agent is **free to choose any tech** for these (Swift,
Rust, Node, Python, Go — whatever fits its environment):

```
adapters/
  discovery/    -> how a fleet is enumerated (mDNS, ARP sweep, ssh-config, a static manifest, k8s)
  transport/    -> how metrics are fetched (ssh, agent/daemon, k8s exec) — MUST return the metrics contract
  render/       -> how the status surface draws (dock icon on macOS, tray, TUI, web tile)
```

## Why this shape

- The **hard-won failure knowledge** lives in `CONTRACTS.md` + `adapters/` notes
  (unified-memory gotcha, SSH-stdin bug, known_hosts race, identity-shadowing),
  so a fresh agent inherits the *lessons* without inheriting the *author's fleet*.
- The agent owns **its** environment; the repo owns the **invariants**.

---

## For the build agent — start here

```
1. READ CONTRACTS.md          — the non-negotiables
2. READ adapters/*            — the seams you implement
3. Author contracts/node.schema.json instantiations for YOUR nodes
4. Implement discovery/transport/render adapters for YOUR stack
5. Validate against contracts/acceptance.md  (the acceptance checklist)
```

`AGENTS.md` is the machine-readable router for exactly this. Go build.
