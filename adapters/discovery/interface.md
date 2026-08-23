# adapter: discovery

**Seam** — the build agent implements how a fleet is enumerated for ITS
environment. The engine must call this seam and consume its output; it must not
care HOW nodes were found.

## Interface
```
discover() -> NodeCandidate[]
```
`NodeCandidate`: `{ name, host, ip, user?, source, reachable?, isDgx? }`

## Output contract
- Never includes the operator's own host (filter by loopback/local IP/local name).
- Positive health filter: for NVIDIA fleets, drop hosts that don't answer
  `nvidia-smi` (ground truth, not a hostname denylist — denylists rot).
- Dedup: collapse by name AND ip; **prefer the entry carrying an SSH identity**
  (config/alias) over a raw-IP broadcast entry (broadcast entries often can't
  authenticate — see lifecycle.md).

## Strategies (implement at least one; know the tradeoffs)
| strategy | pro | con |
|----------|-----|-----|
| mDNS/Bonjour `_ssh._tcp` | zero-config on LAN | misses silent nodes; never lists everything |
| ARP/subnet sweep + TCP22 probe | finds ARP-only hardware | needs warm ARP cache; non-deterministic |
| `~/.ssh/config` (+ includes) sweep | deterministic, identity-aware | only finds what's aliased |
| static manifest | exact, reproducible | operator maintains it |
| k8s/node group (cloud) | dynamic | not for local bare metal |

## Failure lessons (do not re-learn the hard way)
- **Never gate the ARP sweep behind "mDNS found nothing".** If any one node
  broadcasts, the rest (ARP-only) never become candidates. Always merge.
- **Read INCLUDED ssh config files** (e.g. `~/.ssh/config.cluster`) — nodes
  often live there, invisible to a reader that only opens `~/.ssh/config`.
- **Parallel SSH probes race the known_hosts append** → serialize the health
  probe (or use a dedicated known_hosts) or you'll nondeterministically drop nodes.
