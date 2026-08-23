# SecretDGX — CONTRACTS.md (the invariants)

These are the **non-negotiable contracts** every SecretDGX implementation must
satisfy, in any language/stack. If an implementation renegotiates one of these,
it is not SecretDGX. Note where each invariant came from — these are the paid-for
failure lessons, not decorating preferences.

## C1. One config file drives everything
- The entire fleet lives in ONE machine-readable node manifest
  (schema: `contracts/node.schema.json`).
- Nothing environment-specific is compiled or hardcoded into the engine:
  no IPs, no usernames, no node list baked at build time.
- **Lesson:** a per-machine compile (embedding the node list in the binary)
  forces a recompile per environment and is the #1 portability killer.

## C2. Node identity is per-bundle / per-launch, never per-build
- A **single shared binary/runtime** serves any number of nodes.
- Identity is derived from the **launch context** (macOS: bundle identifier
  `...SecretDGX-<name>`; elsewhere: an env var / argv token), not from which
  binary/image was built for which node.
- A node must **never** silently render another node's metrics: on identity
  mismatch, fail loudly (exit), don't guess.
- **Lesson:** whole-run identity shadowing is a silent-catastrophe class bug.

## C3. SSH is the only transport (one short call per node per refresh)
- Metrics come over SSH; exactly **one** connection per node per refresh cycle.
- Every shell loop that SSHs must use `ssh -n` (stdin-redirect). A bare `ssh`
  inside `while read` consumes the loop's pipe and processes only the first node.
- Probe/refresh SSH must **never** mutate the user's real `~/.ssh/known_hosts`
  from a parallel sweep (append race) — use a dedicated file or serialize.
- **Lesson:** SSH-stdin + known_hosts races silently break multi-node loops.

## C4. Lifecycle is repeatable AND reversible
```
discover → authorize → build → run
                       → uninstall (back to genuine fresh-user state, backed up first)
```
- One shared identity (a dock key) authenticates fleet-wide; install only over
  an admin identity the operator already authorizes.
- Uninstall must restore the machine to pre-setup state (config, keys, bundles).

## C5. GB10 unified memory, not VRAM
- On DGX Spark (GB10), `nvidia-smi` VRAM reads **N/A**. MEM% must come from
  **host unified memory** (`/proc/meminfo`, MemTotal/MemAvailable) — the same
  number the NVIDIA dashboard computes.
- **Lesson:** reading VRAM gives you nothing on GB10; this is the top wrong-
  answer trap.

## The metrics contract (what every node reports per refresh)
| field | source | unit |
|-------|--------|------|
| GPU util | `nvidia-smi --query-gpu=utilization.gpu` | % |
| GPU temp | `nvidia-smi --query-gpu=temperature.gpu` | °C |
| power | `nvidia-smi --query-gpu=power.draw` | W |
| CPU | `/proc/stat` delta | % |
| MEM | `/proc/meminfo` (GB10 unified) | % used |
| throttled | `nvidia-smi throttle_reasons.active` | bool |
| online | host reachable (any metric parsed) | bool |

Sub-fields are independently valid: a failed `nvidia-smi` greys its own bar, it
does not kill the whole node.

## Acceptance checklist — `contracts/acceptance.md`
Every implementation must pass it before it may call itself SecretDGX.
