# lifecycle.md — the Lifecycle contract (C4)

The repeatable, reversible lifecycle. Every implementation exposes equivalent
of these operations.

```
┌────────────┐   ┌───────────┐   ┌───────┐   ┌─────┐   ┌───────────┐
│ discover   │ → │ authorize │ → │ build │ → │ run │   │ uninstall │
└────────────┘   └───────────┘   └───────┘   └─────┘   └───────────┘
                                                   (fresh-user reset)
```

## discover
Enumerate candidate nodes from whatever seams the environment offers (mDNS,
ARP subnet sweep, `~/.ssh/config`, a static manifest, k8s). Drop hosts that do
not answer the health probe (for NVIDIA fleets: `nvidia-smi`). Never offer the
operator's own host as a node. Dedupe by name+ip — prefer the entry that carries
an SSH identity (config/alias) over a raw-IP broadcast entry.

## authorize
- Create one shared identity (a dock key: ed25519, no passphrase).
- Install its public half on each authorized node — over the operator's existing
  admin identity, or via password when the node allows it.
- Verify the identity actually authenticates on each node before marking it OK.
- Identity is shared fleet-wide (not per-node), same as the runtime uses.

## build / run
- Produce one launch unit per node from the SAME shared runtime (per C2) —
  pure copy + metadata, no per-node compile.
- Start all of them; each self-identifies via launch context (C2).

## uninstall
- Stop running units.
- Remove generated config, bundles, and identities (backed up first, reversible).
- `--purge-keys` additionally de-authorizes the dock key from every node.
- Net result = genuine first-run state. The pair (uninstall, setup) makes the
  install fully repeatable.

## Key portability rules (violations here are silent-catastrophe class)
| rule | why |
|------|-----|
| `ssh -n` in every `while read` loop | bare `ssh` consumes the loop pipe → 1 node only |
| dedicated known_hosts OR serialize probes | parallel accept-new appends race |
| never embed node list at build | forces per-env compile (C1/C2) |
| identity mismatch → exit, never guess | wrong-node metrics is a silent lie |
