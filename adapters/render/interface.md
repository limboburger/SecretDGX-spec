# adapter: render

**Seam** — the build agent implements how the status surface draws, choosing
whatever fits its OS/desktop. SecretDGX's reference target is a **macOS dock
icon**; the contract is the same for any surface.

## Interface
```
render(nodeName, metrics) -> void    // called each refresh (every ~2s)
```
A render updates the node's surface with:
- node name
- 3 thin bars: **GPU % / CPU % / MEM %** — green/yellow/red
- a distinct **throttle** indicator when `metrics.throttled`
- **offline** state (greyed ✕) when `!metrics.online`
- on select/click → a detail panel with full readout + the ssh alias

## Density rule
Everything visible lives IN the surface — no floating windows/desktop
background for the primary read. One icon per node; detail on interaction.

## Reference target notes (macOS dock)
- macOS owns the Dock tiles — **on-hover of a dock tile isn't capturable**;
  click is the native detail trigger. Don't fight the Dock.
- Launching several windowless dock apps at once races (`open` vs direct exec).
  **Direct-exec** each bundle binary and keep one hidden "keeper" window so
  macOS doesn't auto-terminate after focus loss.
- Positioning the detail panel: locate the owning tile via the Dock window
  (`com.apple.dock`) by bundle title, place the panel just above it.

## Guidelines
- Accessibility first — the surface is a filter on who can use the product.
- Streaming/live: redraw cadence ≈ perceptible-wait threshold (~200ms–2s), not
  a batch dashboard.
- A half-finished surface degrades trust more than an honest one — show real
  offline/unknown states plainly.
