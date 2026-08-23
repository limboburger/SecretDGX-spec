# adapter: transport

**Seam** — the build agent implements how metrics are fetched. Every
implementation must return the **metrics contract** from `CONTRACTS.md`.

## Interface
```
fetchMetrics(node) -> Metrics     // one logical round-trip per node per refresh
```
`Metrics`: `{ online, util %, tempC, powerW, cpu %, mem % (unified), throttled, valid flags }`

## Reference transport: SSH (the one SecretDGX uses)
One short SSH call per node executes the probe and returns delimited values:

```bash
nvidia-smi --query-gpu=utilization.gpu,temperature.gpu,power.draw --format=csv,noheader,nounits
/proc/stat          # CPU delta
/proc/meminfo       # GB10 unified: MemTotal / MemAvailable -> MEM %
nvidia-smi --query-gpu=throttle_reasons.active
```

## Transport rules (C3)
- **Exactly one** connection per node per refresh — batch all fields into one call.
- **`ssh -n`** inside any loop that reads a node list from a pipe (`while read`),
  or the first ssh consumes the rest of the pipe → only node 1 reports.
- Use a **dedicated known_hosts file** (or serialize) for probe/refresh;
  parallel accept-new appends to the operator's real file corrupt it.
- Respect GB10: MEM = host unified via `/proc/meminfo`, never `nvidia-smi` VRAM (N/A).

## Alternatives (agent's choice for its environment)
| strategy | when |
|----------|------|
| SSH (reference) | bare-metal fleet, key auth — default |
| local agent/daemon | nodes you control, no inbound SSH |
| k8s exec / node group | cloud/containerized GPU |

## Per-field validity
Each metric is independently valid: a failed `nvidia-smi` greys its own bars but
does NOT mark the whole node offline. `online` is true if *any* field parsed.
