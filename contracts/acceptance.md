# acceptance.md — the Acceptance checklist

An implementation may call itself SecretDGX only when **all** of these pass in
its target environment. A build agent runs this checklist at the end; a reviewer
(enforcing) uses it as the gate.

## A. Core behavior
- [ ] Every configured node renders one status surface (icon/tile/row).
- [ ] GPU% / CPU% / MEM% update within ~2s.
- [ ] Per-bar color: green < 60, yellow 60–90, red > 90 (or documented equivalent).
- [ ] A clock-throttled GPU shows a distinct throttle indicator.
- [ ] An offline node renders distinctly (greyed ✕) and never crashes the surface.

## B. Identity (C2)
- [ ] A single shared runtime serves 2+ nodes (no per-node build).
- [ ] Launch three nodes from the same binary; each shows ITS OWN metrics.
- [ ] Launch with a bogus node token → clean failure, NO wrong-node metrics.

## C. Config (C1)
- [ ] The full node set is expressed in ONE manifest (per `node.schema.json`).
- [ ] No IP / username is hardcoded in the engine sources.
- [ ] Add a node by editing the manifest only → rebuild/rerun shows it (no code change).

## D. Transport (C3)
- [ ] Exactly one SSH connection per node per refresh.
- [ ] A `while read` loop over the full node set processes ALL nodes (ssh -n).
- [ ] Concurrent probes never corrupt the operator's real known_hosts.

## E. Lifecycle (C4)
- [ ] discover → authorize → build → run completes end-to-end non-interactively.
- [ ] uninstall returns the machine to first-run state; re-setup works (repeatable).
- [ ] A GB10 Spark reports MEM via unified `/proc/meminfo`, not VRAM (C5).

## F. GB10 (C5)
- [ ] On a DGX Spark, MEM is host-unified-memory %, not `nvidia-smi` vram (N/A).

---
Mark every box. If any is unchecked, the implementation is not complete.
