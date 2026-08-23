# identity.md — the Node-identity contract (C2)

The identity rules prevent the worst failure mode in a multi-node status app:
**one node silently rendering another node's metrics.**

## Resolution order
A launch unit resolves its own node name in this order:
1. explicit launch token (`--node=<name>`)
2. launch-context identity (macOS: bundle id `...SecretDGX-<name>`; else: env var)
3. fallback to first configured node (dev only)

## Non-negotiable
- **Mismatch = fatal exit, never a guess.** If the launch token names a node
  that isn't in the manifest, or the bundle id matches no configured node, exit
  with a clear message. Better a dead icon than someone else's numbers.
- **One shared runtime.** All nodes run the same binary/image. Only launch
  context differs. This is what makes add-a-node a no-compile, config-only op.
- **Manifest is the authority.** A launch unit may not render metrics for a node
  absent from the manifest `node.schema.json` set.

## Where identity lives per surface
| surface | identity carrier |
|---------|------------------|
| macOS dock | bundle identifier `local.secretdgx.SecretDGX-<name>` |
| CLI/TUI | `--node=<name>` arg |
| headless/CI | `SECRETDGX_NODE=<name>` env |

## Anti-patterns
- ❌ suffix-matching a bundle id ("...-r1-test" falsely resolves to r1) — require exact match.
- ❌ building one image per node — recreates the per-env-compile trap (C1/C2).
- ❌ letting a raw-IP/identity-less broadcast entry shadow a config entry that
  carries the SSH identity — the config entry must win (see lifecycle.md).
