# lex-cognitive-plasticity

**Level 3 Leaf Documentation**
- **Parent**: `/Users/miverso2/rubymine/legion/extensions-agentic/CLAUDE.md`

## Purpose

Neural plasticity model for adaptive learning. Neural pathways have strength (how established the connection is) and plasticity (how receptive to change). Strengthening increases strength scaled by both amount and current plasticity; plasticity itself decreases (matures) as a pathway is activated — established pathways become harder to change. Critical period mode applies a 2.5× multiplier to all strengthening, modeling sensitive developmental windows. Weak pathways (below `PRUNING_THRESHOLD = 0.15`) are pruning candidates.

## Gem Info

- **Gem name**: `lex-cognitive-plasticity`
- **Module**: `Legion::Extensions::CognitivePlasticity`
- **Version**: `0.1.0`
- **Ruby**: `>= 3.4`
- **License**: MIT

## File Structure

```
lib/legion/extensions/cognitive_plasticity/
  version.rb
  client.rb
  helpers/
    constants.rb
    neural_pathway.rb
    plasticity_engine.rb
  runners/
    cognitive_plasticity.rb
```

## Key Constants

| Constant | Value | Purpose |
|---|---|---|
| `MAX_PATHWAYS` | `300` | Per-engine pathway capacity (auto-prunes weakest on overflow) |
| `MAX_EVENTS` | `500` | Activation event ring buffer |
| `DEFAULT_LEARNING_RATE` | `0.5` | Initial plasticity for new pathways |
| `STRENGTHENING_RATE` | `0.08` | Default amount per strengthen call |
| `PRUNING_THRESHOLD` | `0.15` | Strength below which pathway is a prune candidate |
| `CRITICAL_PERIOD_MULTIPLIER` | `2.5` | Strengthening boost during critical period |
| `MATURATION_RATE` | `0.01` | Plasticity reduction per activation |
| `PLASTICITY_LABELS` | range hash | From `:crystallized` to `:highly_plastic` |
| `STRENGTH_LABELS` | range hash | From `:nascent` to `:robust` |
| `PATHWAY_TYPES` | `%i[synaptic structural functional homeostatic metaplastic]` | Valid pathway types |

## Helpers

### `Helpers::NeuralPathway`
Individual pathway. Has `id`, `label`, `pathway_type`, `strength` (0.0–1.0), `plasticity` (0.1–1.0), and `activation_count`.

- `strengthen!(amount:, critical_period:)` — effective amount = `amount * multiplier * plasticity`; increments `activation_count`; calls `mature!`
- `weaken!(amount:)` — effective amount = `amount * plasticity`
- `prune_candidate?` — `strength < PRUNING_THRESHOLD`
- `mature!` — reduces plasticity by `MATURATION_RATE` (floor 0.1); called automatically on strengthen
- `rejuvenate!(amount:)` — increases plasticity (re-opens the pathway to change)
- `strength_label` / `plasticity_label`
- `to_h`

### `Helpers::PlasticityEngine`
Multi-pathway manager with critical period state.

- `create_pathway(label:, pathway_type:, initial_strength:)` → pathway (auto-prunes weakest if at `MAX_PATHWAYS`)
- `strengthen_pathway(pathway_id:, amount:)` → updated pathway (applies critical_period from engine state)
- `weaken_pathway(pathway_id:, amount:)` → updated pathway
- `rejuvenate_pathway(pathway_id:, amount:)` → updated pathway
- `enter_critical_period!` / `exit_critical_period!` / `critical_period?` — engine-wide mode flag
- `prune_weak_pathways!` → count of pruned pathways
- `pathways_by_type(pathway_type:)` → filtered list
- `strongest_pathways(limit:)` → top N by strength
- `weakest_pathways(limit:)` → bottom N by strength
- `average_strength` / `average_plasticity`
- `prune_candidates_count`
- `plasticity_report` → aggregate stats hash

## Runners

Module: `Runners::CognitivePlasticity`

| Runner Method | Description |
|---|---|
| `create_pathway(label:, pathway_type:, initial_strength:)` | Register a new pathway |
| `strengthen_pathway(pathway_id:, amount:)` | Strengthen (respects critical period) |
| `weaken_pathway(pathway_id:, amount:)` | Weaken a pathway |
| `rejuvenate_pathway(pathway_id:, amount:)` | Re-open pathway to change |
| `enter_critical_period` | Enable 2.5× strengthening multiplier |
| `exit_critical_period` | Disable critical period |
| `prune_weak_pathways` | Remove prune-candidate pathways |
| `strongest_pathways(limit:)` | Top N pathways by strength |
| `plasticity_report` | Aggregate stats |

All runners return `{success: true/false, ...}` hashes.

## Integration Points

- `lex-coldstart` imprint window = critical period: `enter_critical_period` at imprint_window start, `exit_critical_period` at continuous_learning transition
- `lex-memory`: neural pathways are the underlying substrate for memory trace Hebbian links
- `lex-tick` dormant phases: `prune_weak_pathways` is a natural dormant maintenance operation
- `lex-emotion`: high-arousal emotional events can trigger strengthening of associated pathways

## Development Notes

- `Client` instantiates `@default_engine = Helpers::PlasticityEngine.new` via runner memoization
- Effective strengthening = `amount * multiplier * plasticity`: a pathway with plasticity = 0.1 only strengthens at 10% efficiency
- Critical period multiplier (2.5×) is applied by the engine, not the pathway — the engine knows its own state
- `MAX_PATHWAYS = 300` enforces auto-pruning at creation time: weakest pathway is removed to make room
- Plasticity floor is 0.1 (not 0.0): pathways never become completely unmodifiable — they just become very resistant to change
