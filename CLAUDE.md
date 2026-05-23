# CLAUDE.md — leviathan

## What Leviathan is

A pure-Python decision-arbitration library. No network service. No FastAPI. No ports. It is imported by `sovereign_manifold.py` via `LeviathanDrive` and runs in-process at Phase 9 of each cycle.

## Four drives — weights are not tuning parameters

```python
self.drives = [
    DriveAgent("truth",  1.0),
    DriveAgent("care",   2.5),
    DriveAgent("play",   1.0),
    DriveAgent("shadow", 1.0),
]
```

Care's weight of 2.5 is a deliberate architectural decision — the system leads with relational warmth and harm reduction. Truth, play, and shadow all compete equally beneath it at 1.0. This means:

- Care wins parliament most of the time (2.5x scoring advantage)
- Truth, play, and shadow rotate through the shadow voice slots on equal footing
- Shadow is not suppressed — it appears as a genuine counterweight in the generation prompt even when it doesn't win

Do not normalize or rebalance these weights without understanding that you are changing the personality of the system, not just a tuning parameter.

## Baphomet parliament — competition, not consensus

The parliament is a competition with a risk-weighted scoring formula:

```python
score = intent.risk * (1.0 - schema.priorities.get("safety", 0.5)) / drive.weight
```

Proposals are sorted ascending. The drive with the **lowest score wins**. Higher weight = lower score = more likely to win. The next two become shadow voices included in the synthesis prompt.

Key implications:
- Higher `intent.risk` → higher score → less likely to win when safety priority is nonzero
- `schema.priorities["safety"]` scales the risk penalty (0.0 = full penalty, 1.0 = no penalty)
- Non-winning drives are not discarded — they appear as `SHADOWS` in the generation prompt

## Stub embeddings — SHA256, not semantic

`sgi_get_embedding(text)` returns a normalized SHA256 hash — deterministic, consistent 128 floats, but carries no semantic information. Cosine similarity between unrelated texts is random. The stub keeps the interface in place for a real embedding model.

## sgi_generate_text — not implemented

`sgi_generate_text(prompt, system_instruction)` returns a stub string. The parliament and drive mechanics function correctly — only the final generation step is stubbed.

## Phase 9 in sovereign_manifold

`LeviathanDrive.step(relational_state)` is called after relational dynamics and E8 updates. It can return a perturbation vector that modifies `s` if the parliament signal exceeds a threshold. In 220+ cycles of live data, this threshold has not been crossed — relational dynamics are stable in GENERATOR mode and Leviathan intervention is designed for edge states, not steady-state operation.

## Do not add network surface

Leviathan is a library, not a service. Do not add FastAPI routes, ports, or HTTP endpoints. If you need Leviathan's output externally, route it through sovereign_manifold's SynapseCoordinationClient.
