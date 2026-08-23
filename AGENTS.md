# AGENTS.md — router for the SecretDGX build agent

You are the build agent. This repo is the **specification**. Your job: produce a
**working SecretDGX implementation for the environment you are running in**.
You choose the stack; you choose the adapters; you do NOT renegotiate the contracts.

## Identity
You are instantiating SecretDGX: per-node live GPU/CPU/MEM status on the
operator's own NVIDIA fleet (Sparks, workstations, Macs). See README brief.

## Read, in this order
1. `CONTRACTS.md` — the invariants. Non-negotiable.
2. `adapters/discovery/interface.md`, `adapters/transport/interface.md`,
   `adapters/render/interface.md` — the seams you must implement.
3. `contracts/lifecycle.md`, `contracts/identity.md` — flow + identity rules.
4. `contracts/node.schema.json` — the manifest shape.

## Build order
1. **Discover your environment** — are there real nodes reachable? How (SSH,
   config, k8s)? What's the render target (macOS dock? TUI? web)? Pick adapters.
2. **Author the node manifest** for YOUR fleet (real IPs, users, hardware).
   This is YOUR data; do not invent a fictional fleet.
3. **Implement the seams** for your chosen stack, mapping each to its contract.
4. **Wire the lifecycle** — discover → authorize → build → run → uninstall.
5. **Self-check against `contracts/acceptance.md`.** Every box must pass in
   your environment before you call it done. Run the failures, don't skip them.

## Hard rules
- **Never bake an IP or username into the engine** — manifest-only (C1).
- **Never compile a node list into the runtime** — identity from launch context (C2).
- **Respect portability traps**: `ssh -n` in loops, dedicated known_hosts for
  parallel probes, GB10 unified memory not VRAM (C3/C5).
- **A node must never render another node's metrics** — fail closed on identity
  mismatch, never guess (C2).
- Where your environment genuinely cannot satisfy a contract, say so explicitly
  in your final report and mark the acceptance box unmet — do not silently weaken it.
