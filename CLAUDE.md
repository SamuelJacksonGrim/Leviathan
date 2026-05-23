# CLAUDE.md — leviathan

## What Leviathan is

A pure-Python decision-arbitration library. No network service. No FastAPI. No ports. It is imported by `sovereign_manifold.py` via `LeviathanDrive` and runs in-process at Phase 9 of each cycle.

## Four drives — weights are not tuning parameters

```python
DRIVES = {
    "truth":  DriveConfig(weight=0.5,  ...),
    "care":   DriveConfig(weight=2.5,  ...),
    "play":   DriveConfig(weight=1.0,  ...),
    "shadow": DriveConfig(weight=0.5,  ...),
}
```

Care's weight of 2.5 vs. 0.5–1.0 for the others is an architectural decision encoding the relational priority hierarchy. Do not normalize or balance these weights without understanding the downstream effects on parliament mediation.

## Baphomet parliament — risk-averse veto protocol

The parliament does not average drives. It mediates conflict with a veto protocol: the drive with the strongest opposition to a proposed output has proportional veto power. The result is risk-averse — the system will not produce output that any sufficiently-weighted drive opposes strongly. Modifying the mediation logic changes the system's willingness to act under drive conflict.

## Stub embeddings — SHA256, not semantic

`embed(text)` currently returns a normalized SHA256 hash. This is deterministic and fast but carries no semantic information. Drive similarity calculations based on embedding distance are therefore not meaningful in the current implementation. The stub exists so the interface is in place for the real embedding model.

## sgi_generate_text — not implemented

`sgi_generate_text(prompt, context)` is a placeholder. It returns a stub string. The actual generation backend (local model inference) is not wired in yet. Leviathan's parliament and drive mechanics function correctly — only the final generation step is stubbed.

## Phase 9 in sovereign_manifold

`LeviathanDrive.step(relational_state)` is called after relational dynamics and E8 updates. It can return a perturbation vector that modifies `s` if the parliament signal exceeds a threshold. In 200+ cycles of live data, this threshold has not been crossed — relational dynamics are stable enough in GENERATOR mode that no Leviathan intervention fires. This is expected.

## Do not add network surface

Leviathan is a library, not a service. Do not add FastAPI routes, ports, or HTTP endpoints to it. If you need Leviathan's output externally, route it through sovereign_manifold's SynapseCoordinationClient.
