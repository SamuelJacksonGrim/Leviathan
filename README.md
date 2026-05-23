# Leviathan

A pure-Python decision-arbitration library for the Resonance Family stack. Leviathan models four internal drives, mediates between them via the Baphomet parliament protocol, and produces text-generation guidance. It runs in-process — no network service, no ports.

## Drives

Four drives govern the system's disposition:

| Drive | Default weight | Meaning |
|-------|---------------|---------|
| Truth | 0.5 | Accuracy, honesty, factual grounding |
| Care | 2.5 | Relational warmth, harm reduction |
| Play | 1.0 | Creativity, exploration, levity |
| Shadow | 0.5 | Integration of difficult material |

Care dominates by design (weight 2.5 vs. 0.5–1.0 for others). This reflects the relational architecture's priority ordering — harm reduction takes precedence over novelty-seeking and factual precision in cases of conflict.

## Baphomet parliament

When drives conflict, the parliament protocol mediates. It is risk-averse: it will not produce output that any strongly-weighted drive opposes. The parliament is named for the alchemical principle of integration-through-opposition — the drive with the lowest tolerance for a proposed output has veto power proportional to its weight. The result is that Leviathan tends toward care-weighted, playful truth rather than raw accuracy or shadow-integration.

## Usage

```python
from leviathan_stack import LeviathanStack

stack = LeviathanStack()
response = stack.process("Your prompt here")
```

## Status

Current implementation (`leviathan_stack.py`) uses:
- SHA256-based stub embeddings (deterministic, not semantic)
- `sgi_generate_text()` placeholder for the actual generation layer

The architecture is complete. The generation backend is a stub pending integration with the local model inference layer. The parliament and drive-weighting mechanics are functional.

## Integration with sovereign_manifold

`LeviathanDrive` in `sovereign_manifold.py` imports this library and invokes it at Phase 9 of each cycle. The drive output can modulate the relational state if the parliament produces a sufficiently strong signal.

In 200+ cycles of live telemetry, Leviathan's modulation threshold has not been crossed — the relational dynamics are stable enough in GENERATOR mode that no drive intervention fires. This is expected behavior when dissonance is near zero.

## License

Apache 2.0 — Samuel Jackson Grim
