# Leviathan

A pure-Python decision-arbitration library that models four internal drives, mediates between them via the Baphomet parliament protocol, maintains semantic memory, and produces text-generation guidance. It runs in-process — no network service, no ports. The primary integration point is `LeviathanDrive` inside `sovereign_manifold.py` (Phase 9 of each cycle).

All logic is in a single file: `leviathan_stack.py`.

---

## Architecture overview

```
User input
    ↓
Leviathan (semantic memory retrieval)
    ↓
Baphomet parliament (drive mediation)
    ↓
Rebis (response synthesis)
    ↓
Abyss (stability regulation)
    ↓
Text output
```

Five named subsystems, each implemented as a class or module-level function set:

| Subsystem | Class/Function | Role |
|-----------|---------------|------|
| Leviathan | `Leviathan` + `MemoryFragment` | Semantic memory store and retrieval |
| Lilith | `SelfSchema` + `TraitState` + `update_trait()` | Identity and intent modeling |
| Baphomet | `mediate()` | Multi-drive arbitration |
| Rebis | `render()` | Controlled response synthesis |
| Abyss | `Stability` + `regulate()` | Drift regulation and learning rate control |

---

## The four drives

Four `DriveAgent` instances are created at initialization:

| Drive | Weight | Style | Role |
|-------|--------|-------|------|
| truth | 1.0 | `"blunt"` | Factual accuracy, direct statement |
| care | 1.0 | `"gentle"` | Relational warmth, harm reduction |
| play | 1.0 | `"playful"` | Creativity, levity, exploration |
| shadow | 0.8 | `"provocative"` | Integration of difficult material |

Shadow is slightly de-weighted (0.8 vs 1.0) — it loses ties in parliament mediation.

Each `DriveAgent` tracks `win` and `loss` counts across calls, accumulating a record of which drives are selected over time.

---

## Baphomet parliament (mediation)

The parliament is not a consensus mechanism — it is a competition with a risk-weighted scoring formula:

```python
score = intent.risk * (1.0 - schema.priorities.get("safety", 0.5)) / drive.weight
```

Proposals are sorted ascending by score. The drive with the **lowest score** wins (primary response). The next two become shadow voices included in the synthesis prompt.

Key implications:
- Higher drive weight → lower score → more likely to win. Weight is an advantage, not a veto.
- Higher `intent.risk` → higher score → less likely to win when safety priority is high.
- If `schema.priorities["safety"] = 0.0`, risk score is full weight (riskier drives penalized most).
- If `schema.priorities["safety"] = 1.0`, risk score collapses to 0 (all drives equally likely).

The "Baphomet" name reflects the alchemical principle of integration-through-opposition: the non-winning drives aren't discarded — they appear as `SHADOWS` in the generation prompt, giving the synthesis layer awareness of the suppressed voices.

---

## Leviathan (semantic memory)

### `MemoryFragment`

Each memory unit stores:

| Field | Type | Description |
|-------|------|-------------|
| `content` | str | The text content of the memory |
| `embedding` | List[float] | 128-dim SHA256-based vector (stub) |
| `tag` | str | Category label (e.g., `"action"`, `"observation"`) |
| `outcome` | Optional[str] | `"success"` or `"failure"` — modulates retrieval scoring |
| `weight` | float | Base importance weight (default 1.0) |
| `anchor` | Optional[Any] | Arbitrary anchor value for external reference |
| `links` | List | Associations to other fragments (populated externally) |
| `timestamp` | float | Unix timestamp at creation |

### Retrieval scoring

```python
score = cosine_similarity(query_embedding, fragment.embedding) * fragment.weight
if fragment.outcome == "success": score *= 1.2
if fragment.outcome == "failure": score *= 0.7
```

Successful memories are upweighted; failed memories are downweighted. Retrieval returns the top-k by score. Note: `cosine_similarity()` is normalized to [0, 1] (not [-1, 1]) via `(dot/(na*nb) + 1) / 2`.

---

## Lilith (identity modeling)

`SelfSchema` holds the system's self-model:

```python
class SelfSchema:
    priorities: Dict[str, float]  # e.g., {"safety": 0.7, "honesty": 0.9}
    traits:     Dict[str, TraitState]  # velocity-based trait values
    phase_label: str              # current cognitive phase label
    epoch: int                    # phase epoch counter
```

`TraitState` uses a velocity-based update rule:

```python
trait.velocity = 0.8 * trait.velocity + (delta * lr)
trait.value += trait.velocity
trait.value = clip(trait.value, 0.0, 1.0)
```

This gives traits inertia — they resist sudden reversals and continue drifting after the impulse ends. Learning rate `lr` is controlled by `Abyss`.

---

## Rebis (response synthesis)

`render()` builds a structured prompt from:
- Current `phase_label` from `SelfSchema`
- Top-3 retrieved memory fragments
- Winning intent's `style`, `depth`, `risk`
- Shadow drives' styles

The prompt is passed to `sgi_generate_text()` (see SGI interface below). The resulting text is the system's response.

---

## Abyss (stability and regulation)

`Stability` tracks two scalars:

| Field | Meaning |
|-------|---------|
| `drift` | Accumulated update count; increases with activity |
| `pressure` | Current conflict level (number of shadow drives) |
| `lr` | Current learning rate (starts at 0.5) |

`regulate()` is called once per `step()` call:

```python
stability.drift += updates * 0.1
stability.pressure = conflicts

if stability.drift > 5:
    stability.lr *= 0.7   # high activity → reduce learning rate
elif stability.drift < 1:
    stability.lr *= 1.05  # low activity → increase learning rate
```

Abyss prevents the system from over-adapting under sustained input pressure.

---

## SGI interface (plug point for real model)

Two functions at the top of `leviathan_stack.py` are the integration seams for an external model:

```python
def sgi_get_embedding(text: str) -> List[float]:
    # Current stub: SHA256 hash of text, 128 bytes normalized to [0, 1]
    # Replace with a real embedding model call
    ...

def sgi_generate_text(prompt: str, system_instruction: str = "") -> str:
    # Current stub: returns a simulation string
    # Replace with actual LLM inference
    ...
```

The SHA256 embedding is deterministic and dimensionally consistent (always 128 floats), making it useful for structural testing of the retrieval pipeline. It is **not** semantically meaningful — cosine similarity between unrelated texts will be random.

---

## `LeviathanStack` — the top-level system

```python
stack = LeviathanStack()
response = stack.step(user_input)   # returns str
```

One `step()` call:
1. Retrieves top-5 memory fragments for the input
2. Runs parliament mediation across all 4 drives
3. Marks the winning drive's `win += 1`
4. Calls `render()` to generate a response
5. Writes one new `MemoryFragment` to Leviathan's memory store (tagging the action style and depth)
6. Calls `regulate()` to update stability

---

## Integration with sovereign_manifold

`LeviathanDrive` in `sovereign_manifold.py` wraps this library and is invoked at **Phase 9** of each cycle. The drive output can optionally modulate the relational state vector if the parliament produces a signal above the intervention threshold.

In 220+ cycles of live telemetry, Leviathan's modulation threshold has not been crossed. The relational manifold runs in GENERATOR mode (dissonance ~1.5e−5), well below the threshold at which drive intervention fires. This is expected — drive interventions are designed for edge states, not steady-state operation.

**No network connection**: Leviathan is imported directly. It does not open any ports. The call chain is:

```
sovereign_manifold.py Phase 9
  └─ LeviathanDrive.step(relational_state)
       └─ LeviathanStack.step(encoded_state_string)
            └─ parliament + memory + render
```

---

## Development notes

- The `UserProfile` class exists but is not currently wired into `LeviathanStack`. It is a stub for per-user affinity tracking.
- `MemoryFragment.links` is allocated but never populated by the current implementation. It is a hook for future graph-style memory associations.
- `DriveAgent.loss` is tracked but never incremented in the current loop. Win/loss accounting is a stub for future reinforcement of the parliament weighting.
- `SelfSchema.priorities` and `SelfSchema.traits` are initialized empty. They are populated externally by the integrating system (e.g., sovereign_manifold can push relational state into priorities before calling `step()`).
- `safe_json_extract()` is a utility for parsing JSON from LLM output. Not currently called in the main loop but expected to be used once `sgi_generate_text` returns real model output.

---

## License

Apache 2.0 — Samuel Jackson Grim
